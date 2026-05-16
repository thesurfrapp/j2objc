# J2ObjC (Surfr Fork)

Fork of [google/j2objc](https://github.com/google/j2objc) — Google's open-source Java-to-Objective-C transpiler — customized for the Surfr project.

## What Is J2ObjC

J2ObjC is a command-line tool that translates Java source code to Objective-C for iOS and watchOS platforms. It enables Java code to be compiled directly into native Apple frameworks without manual editing of the generated files. The upstream project is documented at [j2objc.org](https://j2objc.org).

J2ObjC supports most Java language features required by client-side applications: exceptions, inner/anonymous classes, generics, threads, and reflection.

## How This Fork Is Used (the `/algorithm` repo)

The [algorithm](https://github.com/thesurfrapp/algorithm) repository contains Surfr's core motion-detection and GPS-tracking algorithms, written in Java (`algo/`). The iOS build in `algorithm/ios/` uses **this fork's `dist/` output** as its transpiler toolchain:

```
J2OBJC_HOME = /Users/herbert/Documents/GitHub/j2objc/dist
```

During an Xcode build of the `SurfrFramework` target, each `.java` file in `algorithm/ios/Algorithm/` is transpiled to Objective-C via a build-phase script:

```bash
${J2OBJC_HOME}/j2objc \
  -d ${DERIVED_FILE_DIR} \
  -sourcepath ${SURFR_ALGO_HOME}/ios/Algorithm/ \
  -classpath ${J2OBJC_HOME}/lib/j2objc_annotations.jar \
  -use-arc --swift-friendly --no-package-directories -g \
  ${INPUT_FILE_PATH}
```

The resulting `.h`/`.m` files are compiled into `SurfrFramework.xcframework` (for iPhone and Apple Watch), which the Surfr frontend app embeds as a linked framework.

### What This Fork Provides

| Artifact | Path | Purpose |
|----------|------|---------|
| `j2objc` CLI | `dist/j2objc` | Transpiles `.java` → `.h` + `.m` |
| JRE runtime | `dist/frameworks/JRE.xcframework` | Provides Java standard library at runtime on iOS/watchOS |
| Annotations JAR | `dist/lib/j2objc_annotations.jar` | Compile-time annotations for controlling translation |
| Headers | `dist/include/` | Objective-C headers for the emulated JRE |

### Why a Fork

This fork adds **watchOS arm64 support** (`watchos64n` architecture) that is not available in upstream J2ObjC. See commit history for the specific patches.

## Building

Prerequisites: JDK 11, Apache Maven, Xcode.

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v11)
export PATH=/opt/apache-maven-3.8.6/bin:$PATH

make clean
make -j8 dist
```

To build only specific architectures (e.g. watchOS):

```bash
export J2OBJC_ARCHS="watchosv7k watchos64"
make -j8 dist
```

Available architectures are listed in `make/common.mk`:
`macosx`, `iphone64`, `iphone64e`, `watchosv7k`, `watchos64`, `watchos64n`, `watchsimulator`, `watchsimulator64`, `simulator`, `simulator64`, `maccatalyst`

### Architecture Notes

- Currently targeting watchOS 8+: use `watchosv7k` + `watchos64`
- When targeting watchOS 9+: swap to `watchos64` + `watchos64n` (drop `watchosv7k`)
- Do NOT build with all three (`watchosv7k` + `watchos64` + `watchos64n`) — the resulting libs become too large

### Important

- **Never** run `make all_dist` or `make dist_protobuf` — protobuf is not used and attempting to build it causes dependency issues.
- The `JRE.xcframework` produced here is also embedded in the frontend project for header resolution.

## Upstream

- **Upstream repo:** https://github.com/google/j2objc
- **Project site:** https://j2objc.org
- **Build guide:** https://developers.google.com/j2objc/guides/building-j2objc (not always accurate)

## Requirements

- JDK 11
- macOS 10.12+
- Xcode 8+
- Apache Maven 3.8+

## License

Distributed under the Apache 2.0 license. See [LICENSE](LICENSE).
