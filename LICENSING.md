# Licensing

This repository is an unofficial fork of [smina](https://sourceforge.net/projects/smina/),
which is itself a fork of [AutoDock Vina 1.1.2](http://vina.scripps.edu/).
It is not affiliated with, endorsed by, or supported by The Scripps Research
Institute, the University of Pittsburgh, or the authors of smina or Vina.

## License of the combined work: GPL-2.0-only

The work as a whole is distributed under the **GNU General Public License,
version 2** (see [`LICENSE`](LICENSE)):

* `src/lib/PDBQTUtilities.cpp` and `src/lib/PDBQTUtilities.h` are derived from
  Open Babel's `pdbformat.cpp` and carry a GPL **version 2 only** notice.
* All four executables link against Open Babel 3, which is GPLv2.

You may therefore use, modify and redistribute this software under the terms of
GPLv2, and any derivative you distribute must be offered under the same terms
with corresponding source.

## Component inventory

| Component | Copyright | License |
| --- | --- | --- |
| AutoDock Vina 1.1.2 core — most of `src/lib`, `src/main` (~70 files carrying an Apache header) | 2006–2010 The Scripps Research Institute; Dr. Oleg Trott, The Olson Lab | Apache-2.0 ([`LICENSE.APACHE`](LICENSE.APACHE)) |
| `src/lib/PDBQTUtilities.{cpp,h}` | 1998–2001 OpenEye Scientific Software; 2003–2006 Geoffrey R. Hutchison; 2004 Chris Morley; 2010 Stuart Armstrong (Source Science/InhibOx); 2014 David Koes and the University of Pittsburgh | GPL-2.0-only ([`LICENSE`](LICENSE)) |
| `src/lib/CommandLine2/` — adapted from LLVM's `llvm/Support/CommandLine` | 2003–2010 University of Illinois at Urbana-Champaign | NCSA / University of Illinois ([`LICENSE.NCSA`](LICENSE.NCSA)) |
| smina's own additions — `src/server`, `src/tosmina`, `src/fromsmina`, `src/pymol`, and the remaining unheadered files in `src/lib` | David Koes / University of Pittsburgh | No per-file notice; distributed as part of the GPLv2 whole |
| Build-system changes in this fork | 2026 Oscar Rojas | Same license as the file modified; see below |

External dependencies are **not** vendored: Boost (Boost Software License 1.0),
Eigen 3 (MPL-2.0), and Open Babel 3 (GPL-2.0) are found on the system at
configure time. An unused in-tree copy of Eigen 3.3.4 that upstream carried in
`src/eigen/` was deleted in this fork.

## Modifications made in this fork

Per Apache-2.0 §4(b) and GPLv2 §2(a), the files changed relative to upstream
smina are listed here. All changes were made in 2026 by Oscar Rojas. They are
build- and portability-only: the docking and scoring code is untouched and
results are unchanged from upstream smina.

| File | Change |
| --- | --- |
| `CMakeLists.txt` | Require C++17; modernize Boost detection (prefer `BoostConfig.cmake`, tolerate header-only Boost.System); use imported targets; find Eigen 3 and Threads as packages; drop forced macOS architectures |
| `CMake/Modules/FindOpenBabel3.cmake` | Add Homebrew search paths for the Open Babel 3 headers and library |
| `src/lib/CommandLine2/CommandLine.cpp` | Replace `boost::unordered_map`/`unordered_set` with the `std` equivalents; qualify `boost::filesystem` calls that became ambiguous against `std::filesystem` |
| `src/lib/file.h` | Replace `boost::filesystem::extension`/`basename` with the modern `path` member API |
| `src/lib/splines.h` | Include Eigen through its installed package path rather than the deleted in-tree copy |
| `src/main/main.cpp` | Drop the removed `boost/filesystem/convenience.hpp` include; replace `boost::filesystem::extension` with `path(...).extension()` |
| `src/server/MinimizationQuery.cpp`, `src/server/QueryManager.{cpp,h}`, `src/server/server.cpp` | Replace `boost::unordered_*` with `std::unordered_*`; use `boost::shared_mutex` consistently; fix a misnamed `boost::unique_lock` variable |
| `src/eigen/`, `src/lib/Eigen` | Deleted — an unused vendored copy of Eigen 3.3.4 and its symlink |
| `src/lib/tmp` | Deleted — a stray duplicate of `weighted_terms.h` |
| `LICENSE.GNU` → `LICENSE` | Renamed to the conventional filename so the references in this file and `README` resolve; the GPLv2 text is unchanged |
| `README`, `.gitignore`, `.gitattributes` | Documentation and repository hygiene |

## Citation

If you use smina, please cite the original paper:

> Koes DR, Baumgartner MP, Camacho CJ. *Lessons Learned in Empirical Scoring
> with smina from the CSAR 2011 Benchmarking Exercise.*
> J Chem Inf Model. 2013. https://pubs.acs.org/doi/abs/10.1021/ci300604z
