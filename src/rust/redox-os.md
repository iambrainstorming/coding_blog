# Redox OS needs to start with  own niche

<https://www.redox-os.org/>

Redox OS needs to create its specialized niche rather than directly competing with Windows, Linux, or Android. It is most likely that developers will be the first to experiment with a new OS. Redox needs to focus on compatibility with RISC-V single-board computers, which can range from desktops to mobile devices, and even Intel or AMD servers. 

The number of Rust packages is growing exponentially. **Transitioning to a new OS has two main challenges: driver and apps.** In five years, when Redox OS matures, most software will be already built with Rust and WASM. RISC-V's open nature solves the problem of driver compatibility.

For serverless computing, there are now WebAssembly serverless options like Spin, which eliminate the need for Docker. This means users can adopt the OS without relying on apps from other languages.

As a simple user like developer, there are a few essential apps that are needed, some of which are already built in Rust: a PDF reader, an EPUB reader, Blender, Inkscape (or Graphite in Rust), GIMP, Helix, Zed, a sound editor (like Audacity), a document editor (like LibreOffice), RustDesk, and OBS Studio.

Most other tasks can be accomplished in a web browser. Servo can be game changer for build ui apps. As a coder, much time is spent in the browser and code editor, so developers are more likely to be open to trying a new OS.

With [kernel 50,000 lines of code](https://doc.redox-os.org/book/ch01-05-how-redox-compares.html#the-kernel) to review and a focus on bug-free, robust security through [sandboxing at the OS level](https://doc.redox-os.org/book/ch04-10-security.html#sandbox), Redox OS has a unique selling point in an era where privacy has become paramount.
