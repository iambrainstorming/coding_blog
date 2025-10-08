# How to call javascript in rust using boa-engine and esbuild?

 ## Folder

 Folder for esbuild

 tests/modules/package.js

 ## Esbuild

 esbuild_tests.js

 ```js
 import { build } from "esbuild";
 import { polyfillNode } from "esbuild-plugin-polyfill-node";

 build({
   entryPoints: ["package.js"],
   bundle: true,
   outfile: "tests/modules/package.js",
   format: "esm",
   // sourcemap: 'inline',
   minify: true,
   plugins: [polyfillNode()],
 }).catch(() => process.exit(1));

 ```

 ## Node packages

 ```bash
 yarn add esbuild
 yarn add esbuild-plugin-polyfill-node
 ```

 ## package.json

 ```json
 "scripts": {
   "esbuild-test": "node esbuild_tests.js"
 }
 ```

 ## Folders for js

 js-api/hello.ts

 ```js
 export function helloWorld(): String {
   return "Hello, world!";
 }
```

package.js

```js
import { helloWorld } from "./js-api/hello";

export { helloWorld };
```



## Example
 https://github.com/boa-dev/boa/blob/main/examples/src/bin/modules.rs


## Imports

```rust
use std::env;
use std::path::PathBuf;
use std::{error::Error, path::Path, rc::Rc};

use boa_engine::{
    builtins::promise::PromiseState, js_string, module::SimpleModuleLoader, Context, JsError,
    JsNativeError, JsValue, Module, NativeFunction,
};
use boa_parser::Source;
```


```rust
 let root = env::current_dir().unwrap();
 println!("Project root: {:?}", root);

 // Build path to package.js
 let js_file_path: PathBuf = root.parent().unwrap().join("tests/modules/package.js");
 println!("JS file path: {:?}", js_file_path);

 let path: &Path = js_file_path.as_path();

 println!("{:?}", path);

 let source = Source::from_filepath(path).unwrap();

 let module_pathbuf = root.parent().unwrap().join("tests/modules");

 let module_path: &Path = module_pathbuf.as_path();

 println!("{:?}", module_path);

 // // println!("source: {:?}", source);
 let loader = Rc::new(SimpleModuleLoader::new(module_path).unwrap());
 // // Instantiate the execution context
 let context = &mut Context::builder()
     .module_loader(loader.clone())
     .build()
     .unwrap(); // // Add the runtime intrisics
 let module = Module::parse(source, None, context).unwrap();

 loader.insert(
     Path::new(module_path)
         .canonicalize()
         .unwrap()
         .join("main.mjs"),
     module.clone(),
 );
 let promise_result = module
     // Initial load that recursively loads the module's dependencies.
     // This returns a `JsPromise` that will be resolved when loading finishes,
     // which allows async loads and async fetches.
     .load(context)
     .then(
         Some(
             NativeFunction::from_copy_closure_with_captures(
                 |_, _, module, context| {
                     // After loading, link all modules by resolving the imports
                     // and exports on the full module graph, initializing module
                     // environments. This returns a plain `Err` since all modules
                     // must link at the same time.
                     module.link(context)?;
                     Ok(JsValue::undefined())
                 },
                 module.clone(),
             )
             .to_js_function(context.realm()),
         ),
         None,
         context,
     )
     .then(
         Some(
             NativeFunction::from_copy_closure_with_captures(
                 // Finally, evaluate the root module.
                 // This returns a `JsPromise` since a module could have
                 // top-level await statements, which defers module execution to the
                 // job queue.
                 |_, _, module, context| Ok(module.evaluate(context).into()),
                 module.clone(),
             )
             .to_js_function(context.realm()),
         ),
         None,
         context,
     );

 // Very important to push forward the job queue after queueing promises.
 context.run_jobs();

 match promise_result.state() {
     PromiseState::Pending => {
         println!("Module can't load");
     }
     PromiseState::Fulfilled(v) => {
         assert_eq!(v, JsValue::undefined());
     }
     PromiseState::Rejected(err) => {
         println!(
             "{:?}",
             JsError::from_opaque(err).try_native(context).unwrap()
         );
     }
 }

 // We can access the full namespace of the module with all its exports.
 let namespace = module.namespace(context);

 let hello_world_namespace = namespace.get(js_string!("helloWorld"), context).unwrap();
 let hello_world = hello_world_namespace
     .as_callable()
     .ok_or_else(|| JsNativeError::typ().with_message("mix export wasn't a function!"))
     .unwrap();
 let result = hello_world
     .call(&JsValue::undefined(), &[], context)
     .unwrap();

 println!("result = {}", result.display());

 ```

 ```bash
cargo test -- --nocapture
```
