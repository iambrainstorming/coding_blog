# Parsing with Xpath in rust

You can use package: sxd-xpath

```rust
cargo new xml_parse
```

In Cargo.toml

```
[dependencies]
sxd-xpath="0.4.2"
sxd-document="0.3.2"
```

```rust
use std::fs;
use sxd_document::parser;
use sxd_xpath::{evaluate_xpath, Context, Factory, Value};
fn remove_first_two_lines(input: &str) -> String {
    let mut lines = input.lines();
    let mut result = String::new();

    // Skip the first two lines
    lines.nth(0);
    lines.nth(0);

    for line in lines {
        result.push_str(line);
        result.push('\n');
    }

    result
}

fn get_list() {
    let file = fs::read_to_string("src/ids.xml").unwrap();
    let data = remove_first_two_lines(&file);
    let package = parser::parse(&data).expect("failed to parse XML");
    let document = package.as_document();

    let value = evaluate_xpath(&document, "//Id");

    match value {
        Ok(Value::Nodeset(nodeset)) => {
            for node in nodeset.iter() {
                println!("{}", node.string_value());
            }
        }
        _ => {
            println!("XPath expression didn't return a nodeset");
        }
    }
}

fn main() {
    get_list();
}
```


ids.xml
```rust
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eSearchResult PUBLIC "-//NLM//DTD esearch 20060628//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20060628/esearch.dtd">
<div>
<Id>37793170</Id>
<Id>37793014</Id>
<Id>37792757</Id>
<Id>37792561</Id>
<Id>37792179</Id>
<Id>37790950</Id>
<Id>37790689</Id>
<Id>37790685</Id>
<Id>37790681</Id>
<Id>37790651</Id>
<Id>37789937</Id>
<Id>37789815</Id>
<Id>37789674</Id>
<Id>37789431</Id>
<Id>37788598</Id>
<Id>37788081</Id>
<Id>37787682</Id>
<Id>37787533</Id>
<Id>37787338</Id>
<Id>37787330</Id>
<Id>37787327</Id>
<Id>37787268</Id>
<Id>37783146</Id>
<Id>37781996</Id>
<Id>37781611</Id>
<Id>37779775</Id>
<Id>37779596</Id>
<Id>37779203</Id>
<Id>37777423</Id>
<Id>37776597</Id>
<Id>37776007</Id>
<Id>37774586</Id>
<Id>37774576</Id>
<Id>37773778</Id>
<Id>37773541</Id>
<Id>37772835</Id>
<Id>37772621</Id>
<Id>37771545</Id>
<Id>37771101</Id>
<Id>37770849</Id>
<Id>37770282</Id>
<Id>37768948</Id>
<Id>37768533</Id>
<Id>37768482</Id>
<Id>37768298</Id>
<Id>37766883</Id>
<Id>37766376</Id>
<Id>37766316</Id>
<Id>37766298</Id>
<Id>37765198</Id>
</div>
```


