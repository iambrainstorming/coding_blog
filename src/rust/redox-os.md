# Redox OS needs to start with  own niche

<https://www.redox-os.org/>

Redox OS needs to create its specialized niche rather than directly competing with Windows, Linux, or Android. It is most likely that developers will be the first to experiment with a new OS. Redox needs to focus on compatibility with RISC-V single-board computers, which can range from desktops to mobile devices, and even Intel or AMD servers.

## Two main challenges
**Transitioning to a new OS has two main challenges: drivers and apps.** In five years, when Redox OS matures, most software will be already built with Rust and WASM. The number of Rust packages is growing exponentially. RISC-V's open nature solves the problem of driver compatibility.

## Serverless computing
For serverless computing, there are now WebAssembly serverless options like [Spin](https://github.com/fermyon/spin), which eliminate the need for Docker. This means users can adopt the OS using webassembly.

## App ecosystem
As a simple user like developer, there are a few essential apps that are needed, some of which are already built in Rust: a [PDF reader](https://github.com/pdf-rs/pdf), an EPUB reader, Blender, Inkscape (or Graphite in Rust), GIMP, [Helix](https://helix-editor.com/), [Zed](https://zed.dev/), a sound editor (like Audacity), a document editor (like LibreOffice), [RustDesk](https://github.com/rustdesk/rustdesk), and OBS Studio.

Compatibility with [Tauri for apps](https://tauri.app/) can help to grow the Redox OS app ecosystem.

## Rust browser
Most other tasks can be accomplished in a web browser. Servo can be game changer for building ui apps with ease, also at OS level UIs. Linux took a long time to achieve mass adoption due to its historically poor UI design. As a coder, much time is spent in the browser and code editor, so developers are more likely to be open to trying a new OS.


## Security

With [kernel 50,000 lines of code](https://doc.redox-os.org/book/ch01-05-how-redox-compares.html#the-kernel) to review and a focus on bug-free, robust security through [sandboxing at the OS level](https://doc.redox-os.org/book/ch04-10-security.html#sandbox), Redox OS has a unique selling point in an era where privacy has become paramount.

Sandboxing for non-root users needs to be easy to maintain high security. Even if a user installs malicious apps downloaded from the internet, your computer should remain safe. Sandboxing should be highly customizable, with the ability to control internet access, read and write permissions for files and folders, and specify which folders an app can access. It should also be easy to manage through both a user interface and command-line interface.

## Rust and Redox OS: The Future of Efficient Computing

With [Moore's Law coming to an end](https://iambrainstorming.github.io/chapters/computer/moore-law.html), the speed and efficiency of software will become a paramount issue. The Rust language offers performance comparable to C and C++, while its rich type system and ownership model guarantee memory safety and thread safety, features that are not provided by C and C++. This enables the elimination of many classes of bugs at compile-time, making Rust a more efficient language than C and C++. 

The Redox OS, built on Rust,  will inherit these features, boasting faster boot times of mere seconds and a responsive operating system, even on low-end devices such as those with Celeron processors or single-board computers like the Raspberry Pi.

The microkernel design of Redox OS, with its minimal lines of code, will result in a smaller-sized OS, yielding a significantly lower carbon footprint, even lower than that of Linux.

## About Licence

Redox uses MIT licence rather than [AGPL or GPL](https://iambrainstorming.github.io/chapters/programming/foss_philosophy.html#which-open-source-license-to-use). With MIT licence corporations have no incentive to open-source their changes to the code. This could lead to a problem where big tech companies hijack the code without contributing back to the open-access development of the OS. For complex operating system, a GPL license might be suitable, but its my personal opinion. GNU copyleft licenses may have particularly helped Linux protect its code and prevent corporations from converting it to closed source during its early days. With Redox OS, things are different. Redox is a microkernel with a manageable number of lines of code, and each driver, filesystem, network protocol, and more is provided as a separate process. This means that each component has full licensing autonomy and is not legally bound by the license of other OS code, including the microkernel license. However, it's important to note that no open-source license, including the GPL, can completely prevent big tech companies from converting the code to proprietary software. They can copy the design and change function names within a short period, and such changes may not be easily detected by plagiarism checkers, especially with the help of AI translators. The microkernel has fewer lines of code than the Linux kernel, making it easier to copy.

Many users choose to use FOSS not only because it solves their problems but also because it provides them `freedom`. [Freedom](https://iambrainstorming.github.io/chapters/programming/foss_philosophy.html) is an important value sought by many FOSS users and developers, who prefer to use and develop software that respects their freedoms. For example, I prefer to use only FOSS applications if there are alternatives available. In such cases, the type of FOSS license becomes less relevant, as the primary concern is having the freedom to use, modify, and distribute the software as desired.

## Redox OS Foundation

As Redox OS is composed of multiple parts with different licenses as a FOSS user, am concerned about the potential inclusion of proprietary components. My preference would be for the OS to be entirely free and open-source, with no proprietary elements included. This would depend on the philosophy of the foundation that maintains the assembly, similar to the Ubuntu Foundation. I would place my trust in such a foundation to uphold these values and maintain the integrity of the open-source community.
