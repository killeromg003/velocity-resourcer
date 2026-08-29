# velocity-resourcer

A small C++ tool to collect and pack game assets (fonts, images, icons, particles, weather effects, etc.) into a single binary resource package (.rpak) used by the Velocity engine/tooling.

I inspected the project's Visual C++ project files and key sources (project/entry.cpp, project/core/rpack/ids.hpp, and project/core/rpack/impl) to produce this README.

## Features

- Packs multiple resource types (fonts, images, icons, particles, particle effects, weather sets, etc.) into a single .rpak file.
- Includes a compact resource ID system (see project/core/rpack/ids.hpp).
- Reader/writer implementation for the custom rpack format (project/core/rpack/impl/reader.cpp, writer.cpp).
- Uses a simple entry program (project/entry.cpp) which maps filesystem assets to resource IDs and writes the .rpak file.

## Quick overview

- Language: C++ (Visual C++ / MSVC project)
- Purpose: Create a resources package file (resources.rpak) from asset files in the repository
- Main components:
  - project/entry.cpp — the application that registers assets and invokes the packer
  - project/core/rpack/* — resource ID definitions and rpack reader/writer implementation
  - project/core/resources/* — directories expected to contain source assets (fonts, images, particles, etc.)
  - project/pch/* — precompiled-header support for the MSVC build

## Repository layout

```
project/
  entry.cpp                     # main program — builds and saves resources.rpak
  pch/
    stdafx.cpp
    stdafx.hpp                  # precompiled header files
  core/
    rpack/
      ids.hpp                   # resource ID enums/namespaces
      rpack.hpp                 # rpack public API
      impl/
        reader.cpp              # rpack reading implementation
        writer.cpp              # rpack writing implementation
    resources/
      fonts/                    # font files referenced by ids.hpp / entry.cpp
      images/                   # image files referenced by ids.hpp / entry.cpp
      icons/                    # icon images
      particles/                # particle definitions / effects / assets
      weather/                  # weather-related assets (rain, snow, etc.)
      ...                       # other asset subfolders used by entry.cpp
velocity-resourcer.vcxproj      # MSVC project file
velocity-resourcer.vcxproj.*    # MSVC project metadata/filters/user settings
```

## How it fits together

The entry program (project/entry.cpp) constructs a writer, maps in-memory resource IDs (from project/core/rpack/ids.hpp) to files under project/core/resources/*, then calls the rpack writer to serialize everything to a single resources.rpak file. The rpack impl (reader/writer) provides the packing format and the code that reads and writes the binary package.

## Build & run

Prerequisites:
- Windows with Visual Studio (MSVC). The repository contains an MSVC project file (.vcxproj).
- Place your asset files under the expected directories in project/core/resources/ following the names used in project/core/rpack/ids.hpp and project/entry.cpp.

Build with Visual Studio:
1. Open `velocity-resourcer.vcxproj` in Visual Studio and build the project (choose Debug or Release).
2. Run the produced executable (from Visual Studio or the output directory). By default the executable will look for assets in project/core/resources and produce `resources.rpak` in the working directory.

Build from the command line (MSBuild):
```powershell
msbuild velocity-resourcer.vcxproj /p:Configuration=Release
# Then run the built executable (path depends on your MSBuild configuration)
# e.g. .\Release\velocity-resourcer.exe
```

If the program cannot find expected resources it will print an error; ensure the resources directory structure matches the IDs used in `project/entry.cpp` and `project/core/rpack/ids.hpp`.

## Usage notes

- The mappings between resource IDs and filesystem assets are defined in `project/entry.cpp`. To add or change assets:
  - Add files into the appropriate subfolder under `project/core/resources/`.
  - Update `project/core/rpack/ids.hpp` if you need new resource IDs (follow the existing namespaces and enums).
  - Update `project/entry.cpp` to add the new mapping using `writer.add(<rpack::...::id>, <resources::...::path>);`.
  - Rebuild and run to produce an updated `resources.rpak`.

- The `rpack` implementation is intentionally minimal and focused on the binary packing format used by the project. Consult `project/core/rpack/rpack.hpp` and the impl sources for extension points.

## Extending & contributing

- Add new resource categories in `ids.hpp` (namespaces like `fonts`, `images`, `particles`, etc.).
- Keep `entry.cpp` in sync with `ids.hpp` and the assets under `project/core/resources/`.
- If you add large or binary assets, consider .gitignore or LFS to avoid bloating the git repository.

## Limitations & missing pieces

- There is no LICENSE file in the repository — add one if you intend to make the project public/open-source with licensing terms.
- No automated tests or CI workflows are included.
- The project assumes MSVC/Visual Studio; cross-platform builds are not provided.

## Contact / Author

Repository maintained by the original author in the repository. Open issues or PRs on GitHub for questions, bug reports, or contribution proposals.