# Paradoc

> A modern, native DICOM toolkit for Linux — built in C#/.NET on top of [fo-dicom](https://github.com/fo-dicom/fo-dicom). Its first two applications are **Batavia** (a mini-PACS server) and **Rebus** (a viewer).

*(The names of the framework and its components are provisional and may be changed in the future).*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: early development](https://img.shields.io/badge/status-early%20development-orange)]()

## Why this project

DICOM tooling on Linux has a real gap. [Weasis](https://github.com/nroduit/Weasis) is open-source and cross-platform but built on Java/Swing. [OHIF](https://ohif.org/) is an excellent web viewer but needs a DICOMweb-compliant server behind it. [Orthanc](https://www.orthanc-server.com/) is a strong lightweight PACS server but not a full desktop viewing client. There is currently no actively developed, native, modern-toolkit desktop DICOM viewer for Linux comparable to what Windows users have with tools like MicroDicom or RadiAnt.

[fo-dicom](https://github.com/fo-dicom/fo-dicom) provides a solid, actively maintained .NET implementation of the DICOM standard. What's missing is the application layer above it: reusable, well-designed building blocks (patient/study/series management, rendering, storage, networking) that a Dicom client (such as a viewer), a PACS server, or specified tools can all share. **Paradoc** is designed to play the role of this layer.

**This project is not "yet another DICOM viewer."** Paradoc is a framework — with a viewer (**Rebus**) and a mini-PACS server (**Batavia**), as its first concrete, shippable applications. DICOMweb/FHIR interoperability are a longer-term direction once the core is solid.

## Project goals

- **Paradoc: a DICOM framework for .NET/Avalonia** — reusable components and abstractions built upon fo-dicom, usable independently of any single application.
- **A desktop viewer application (Rebus)** — native, fast, Linux-first (cross-platform via Avalonia), MIT-licensed.
- **A mini-PACS server application (Batavia)** —classical Dicom networking.
- **Longer-term**: DICOMweb and FHIR extension points, once the core framework and both flagship apps are stable.

*Not a goal, at least initially*: replacing clinical-grade certified PACS/viewer systems. This project does not currently target regulatory clearance for primary diagnostic use — see [Scope & Disclaimer](#scope--disclaimer).

## Tech stack

| Layer | Choice |
|---|---|
| Language / runtime | C# / .NET 9+ |
| DICOM protocol & data model | [fo-dicom](https://github.com/fo-dicom/fo-dicom) |
| UI framework | [Avalonia UI](https://avaloniaui.net/)  |
| Metadata storage | SQLite |
| Image storage | Structured filesystem, UID-addressed (see [Storage architecture](#storage-architecture)) |
| Target platform | Linux-first, cross-platform by virtue of Avalonia + .NET |
| License | MIT |

## Storage architecture

- **Metadata** (patient/study/series/instance hierarchy) lives in a local **SQLite** database. SQLite was chosen over alternatives like LiteDB specifically because the viewer and server are separate processes that may access the same data concurrently — The DB access layer will be abstracted behind an interface so this choice can be revisited without touching application code.
- **Dicom files with Pixel data** live on the filesystem, organized by DICOM UID rather than human-readable identifiers:
  ```
  storage/<PatientID>/<StudyInstanceUID>/<SeriesInstanceUID>/<SOPInstanceUID>.dcm
  ```

- **Caching**: two separate caches are planned — an in-memory LRU cache for decoded pixel/frame data, and a thumbnail cache for fast series-browsing.

## Current status

**Early development / pre-alpha.** This project started as a discussion among a small group interested in DICOM and .NET tooling on Linux. There is no released software yet. We are currently working toward a first runnable milestone — see below.

## First milestone (concrete, scoped)

To avoid the classic "let's design the whole framework first" trap, the first milestone is deliberately narrow — and deliberately *not* trying to be a usable application yet:

> **M1: A minimal Avalonia application that opens a local DICOM file, decodes it via fo-dicom, and displays the pixel data in a window. No window/level, zoom, or pan, and no persistence — the file is opened, shown, and forgotten on exit. Just correct decode-and-display.**

Planned milestones after M1 (subject to change as the project takes shape):

- **M2**: Storage module — SQLite metadata schema, UID-addressed filesystem image storage, and the pixel/thumbnail caching layer described in [Storage architecture](#storage-architecture). No UI yet; provable via unit tests and a minimal CLI. This is the foundation M3 builds on, and a good self-contained task for a contributor who isn't yet working with Avalonia.
- **M3**: Local patient/study/series management UI — browse, create, edit, delete - built upon the M2 storage module. Includes a tag-rewrite engine with correct UID regeneration (SOPInstanceUID/StudyInstanceUID/SeriesInstanceUID cascade on edits). "Refurbishing" an existing series (relabeling patient/study/series tags, keeping pixel data) is the primary use case exercising this engine.
- **M4**: Creation of series through "refurbishing" collections of existing Dicom files by using their pixel data and re-writing all relevant tags. Creation of Dicom objects through **Secondary capture**.
- **M5**: Interactive viewer features — window/level, zoom, pan, thumbnail/series browsing UI.
- **M6**: Classical Dicom networking (C-ECHO, C-STORE, C-MOVE, C-FIND) — both for the viewer and the mini-PACS server.
- *(Further out)*: DICOMweb (WADO-RS/QIDO-RS), FHIR interop.

## Getting involved

This project is genuinely early. If you get involved now, you'll shape the architecture, not just consume it.

Useful backgrounds:
- C#/.NET development
- Experience with Dicom frameworks (fo-dicom, DicomObjects, DCMTK, dcm4che, pydicom, etc.)
- Avalonia, WPF, or other XAML experience
- Medical imaging domain knowledge (radiology, PACS administration, imaging informatics)
- Interest in DICOMweb/FHIR, even if not immediately relevant

**How to start:**
1. Read this README and skim the [open issues](../../issues) — look for the `good first issue` label once they exist.
2. Introduce yourself in [Discussions](../../issues) — tell us what you're interested in working on.
3. For anything non-trivial, open an issue to discuss the approach before submitting a PR — this is a young project and we're still converging on conventions.

## Scope & disclaimer

This software is not certified as a medical device and is not intended for primary diagnostic use. It is intended for research, education, workflow tooling, and as a foundation for further development. Contributors and users working toward clinical deployment should independently evaluate applicable regulatory requirements (e.g. FDA, EU MDR) for their jurisdiction and use case.

## License

MIT — see [LICENSE](LICENSE). Deliberately chosen to keep this usable in both open and commercial downstream projects without friction.

## Contact / discussion

*(Add: Codeberg repo URL, any chat/forum link once set up, maintainer contact)*

---

*This README is itself a draft — feedback on framing, scope, and milestones is welcome, especially before we start attracting contributors around it.*
