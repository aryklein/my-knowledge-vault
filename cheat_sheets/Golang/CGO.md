---
tags:
  - golang
---
# CGO

CGO is a feature in Go (Golang) that allows Go programs to call C code. This
integration is crucial for cases where you need to use libraries written in C or
for some system-level operations that are only accessible via C APIs. The CGO
programming facility provides the mechanism to both call C functions and to
manipulate C types directly from Go code.

When you compile your Go code with the command:
```bash
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o myapp .
```

You are creating a binary that is:

- Completely statically linked (no dynamic linking to C libraries).
- Compiled for Linux, regardless of your development OS.
- Ensuring a clean build without relying on potentially outdated dependencies.

You are instructing the Go compiler to build your application with specific
settings and constraints. Here's what each part of the command means:


1) `GO_ENABLED=0`: This environment variable disables CGO. When CGO is disabled,
the Go compiler will not compile any C code in your program (including the C
parts of the standard library). This is often done to create a statically linked
binary that does not depend on the system's C libraries, making the binary more
portable across different Linux distributions.

2) `GOOS=linux`: This sets the target operating system to Linux. The Go compiler
will build the binary so that it can run on Linux systems, regardless of the OS
you're using to compile the code. This is part of Go's cross-compilation
support.

3) `go build`: This is the command to compile the Go program.

4) `-a`: This flag forces rebuilding of all packages that the application
depends on. This is useful to ensure that you're not using any old, cached
versions of packages, especially when you change build tags or other settings
like `CGO_ENABLED`.

5) `-installsuffix cgo`: This flag adds a suffix to the package installation
path, effectively separating the build cache for your non-CGO build from other
builds. It's particularly useful to avoid conflicts and cache issues when you
switch between CGO and non-CGO builds.

6) `-o main`: Specifies the name of the compiled binary. In this case, the
binary will be named main.

7) `.`: This specifies the current directory as the package to build.
