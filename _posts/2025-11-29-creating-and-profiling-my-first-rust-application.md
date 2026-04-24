## Introduction
This blog post is a rewrite of my university assignment where I had to parallelise a synchronous program. It goes over a lot more than I described in my assignment, going into the code itself and the struggles faced as well rather than just the parallelisation results.

The project being parallelised in this blog post is called BadRust. The purpose of this program is to take any .mp4 video file and play the video in the terminal using ASCII characters. The video input for these types of projects is popular internet video [Bad Apple](https://www.youtube.com/watch?v=FtutLA63Cp8) as the binary black and white characteristics of the video makes it easy to show on any device that can output pixels. This has led to unique projects that use BadApple to showcase video processing capabilities such as a video from Junferno that showcases Bad Apple being played on Deimos.

### Why Rust?
The reason why the project was written in Rust was due to three reasons. 
- None of the allocated projects for the assignment could run on my computer since they depended on C# GUI libraries made for Windows or used very old versions of programming languages that would make researching solutions to parallelise software difficult. 
- After reading some documentation on [multi-threading](https://doc.rust-lang.org/book/ch16-01-threads.html) the approach Rust took felt approachable and I felt like I was in good hands due to the very detailed documentation. 
- This presented a good opportunity to expand my skills to other programming languages. I always wanted to improve my skills in lower level languages but never had a good excuse for it and this seemed to be the perfect opportunity.

As previously described BadRust is written in rust. It is also a rewrite of one of my previous projects called BadApple. BadApple was written in python many years ago back in high school and during this project served as a template on how to create the synchronous version of this application in Rust. During the planning phase of writing this software I was considering just parallelising the python implementation instead of rewriting everything. This is not favourable due to python's global interpreter lock (GIL)  

> *"The python GIL synchronises the execution of threads so that only one native thread (per process) can execute basic operations"*[^1]

This prevents the python interpreter from running multi-threaded. There might be ways around this factor but due to time constraints and that there is no bonus points for going down that I decided to rewrite the program in another language that would give me more granularity in controlling the threads while also being quite safe.

### Tooling
I wanted to use [Perf](https://en.wikipedia.org/wiki/Perf_(Linux)) with [FlameGraph](https://github.com/brendangregg/FlameGraph) for this assignment. Perf is included in the Linux Kernel and FlameGraph to capture Perf stacks. I have experience with [Intel VTune Profiler](https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler-download.html) but it was very hard to use with Arch and is one of the applications I had to use with distrobox as I found it troubling getting it running on my host system.

Another thing to state is [rust-analyzer](https://rust-analyzer.github.io/) the Rust language server. It was very easy to install using [Mason](https://github.com/mason-org/mason.nvim). The language server is needed since I used [LazyVim](https://www.lazyvim.org/) to program this project and it allows those auto complete options to show up while coding and linting errors all in your editor.

## Project Definition





## Project Definition


The project heavily relies on [ffmpeg](https://ffmpeg.org/) (which is parallelised by default) to convert the frames of the video to a folder for further processing. This means in order for the project to run correctly ffmpeg needs to be installed on the host machine and added to the operating system’s PATH variable in order to run.


- Image to ASCII implementation from img2ascii project
- Arguments
- Rayon used for parallelisation
- FFmpeg for frame to image conversion

## Analysis of Potential Parallelism
Since the program uses the Rayon library, parallelism can be found by finding iterable objects and converting them to parallel iterators. Therefore, one area of concern arose at that would be the for loop that iterates over each frame and converts them to ASCII frames. There is also the section where the images are converted to grayscale but parallelising that would prove no significant speedup as the outer loop is already parallelised. Despite this this part of the program was parallelised anyways and led to some interesting results.

Perf was used to try to identify places where parallelisation could be achieved but instead found something else which will be discussed later in overcoming performance problems.

## Mapping Computation to Processors

## Timing and Profiling Results

## Producing the Same Result

## Tools and Techniques

## Overcoming Performance Problems

## Modified Code

## GPU Parallelisation
Due to time constraints I wasn't able to implement GPU parallelisation.
## Conclusion

[^1]: https://en.wikipedia.org/wiki/Global_interpreter_lock
