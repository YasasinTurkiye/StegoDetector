Stego Detector is a Windows desktop app that scans a single file or a whole
folder (including all subfolders) for signs that data has been hidden inside
files, and shows you what it found and why.

It only LOOKS. It never changes, cleans or repairs files, never extracts or
saves hidden payloads, and never tries to crack passwords. Everything runs
on your own PC - nothing is uploaded, and the app makes no network
connections.


--------------------------------------------------------------------------------
 CONTENTS
--------------------------------------------------------------------------------
  1. Requirements
  2. Getting started
  3. Using the app
  4. Understanding the results
  5. What gets checked
  6. Options
  7. Exporting reports
  8. Command-line version
  9. Limitations - please read
 10. Troubleshooting
 11. Building from source
 12. Project structure


--------------------------------------------------------------------------------
 1. REQUIREMENTS
--------------------------------------------------------------------------------
  - Windows 10 or Windows 11 (64-bit)
  - .NET 9 Desktop Runtime (x64) - free from Microsoft:
      https://dotnet.microsoft.com/download/dotnet/9.0
    (choose ".NET Desktop Runtime" for Windows x64)

  If the runtime is missing, Windows shows a message with a download link
  the first time you start the app. You only need to install it once.


--------------------------------------------------------------------------------
 2. GETTING STARTED
--------------------------------------------------------------------------------
  1. Download StegoDetector.exe (single file, about 0.6 MB).
  2. Put it anywhere you like - no installation needed.
  3. Double-click StegoDetector.exe.


--------------------------------------------------------------------------------
 3. USING THE APP
--------------------------------------------------------------------------------
  The window has four pages in the left sidebar: Home, Scan, Results, About.

  HOME - choose what to scan
    Pick a target in any of these ways:
      - Click "Browse..." and choose "Select file..." or "Select folder..."
      - Type or paste a path into the box
      - Drag a file or folder onto the window
      - Copy a file in Explorer and press Ctrl+V in the app
    Then click "Start Scan". (The button stays grey until the path exists.)

  SCAN - live progress
    Shows the file being analysed, files done / total, a progress bar,
    counters per confidence level, and detections as they are found.
    "Cancel" stops the scan; results found so far are kept.

  RESULTS - the full report
    - One row per finding. Files with nothing found appear as CLEAN,
      files that could not be read appear as ERROR.
    - Click a column header to sort.
    - Type in the search box to filter (path, type, method, tool, evidence).
    - Use the High / Medium / Low / Errors / Clean buttons to show or hide
      each level.
    - Select a row to see its full evidence at the bottom, including a
      read-only hex/text preview of the suspicious bytes.
    - "Copy" copies the evidence text; "Show in folder" opens Explorer.

  ABOUT - version and a short explanation of the confidence levels.


--------------------------------------------------------------------------------
 4. UNDERSTANDING THE RESULTS
--------------------------------------------------------------------------------
  Every finding has a confidence level. It describes how strong the evidence
  is - it is not a verdict.

  HIGH    Concrete evidence. Examples: a complete file embedded inside
          another file; data stored after the point where the file format
          ends; a known tool's fingerprint (e.g. the F5 encoder comment, the
          OpenStego header, readable text in the lowest bits of an image).

  MEDIUM  A statistical anomaly. Examples: several pixel-statistics tests
          agree that the lowest bits look modified; a JPEG whose DCT
          statistics look manipulated; a block of random-looking
          (encrypted?) data where the format doesn't normally have it;
          a metadata field stuffed with binary or encoded data.

  LOW     A single weak hint. Often harmless - icons, computer-generated
          graphics and dithered GIFs can trigger these. Only worth a closer
          look together with other evidence.

  ERROR   The file could not be read or is corrupt / truncated. Errors are
          never counted as clean. Checks that don't depend on the format
          (such as appended data) may still report findings.

  CLEAN   None of the checks that apply to this file type found anything.
          Note: no tool can prove a file contains nothing hidden.

  JPEG findings also show:
    Tool      the suspected tool (JSteg, OutGuess, JPHide, F5, ...) or
              "DCT anomaly, tool unknown"
    Rating    * / ** / *** (like stegdetect) - more stars = stronger
    Score     the test value that produced the finding
    Sensitivity  the sensitivity setting used for the scan

  The file type shown is the REAL type, detected from the file's content.
  A renamed file (e.g. a ZIP saved as "holiday.jpg") is analysed as what it
  really is, and the mismatch is reported.


--------------------------------------------------------------------------------
 5. WHAT GETS CHECKED
--------------------------------------------------------------------------------
  EVERY FILE
    - Real file type by its header ("magic bytes"), not by extension
    - Data appended after the format's end marker (with offset, size, type
      of the appended data and a preview)
    - Files hidden inside other files (ZIP, RAR, 7z, PDF, PNG, JPEG, GIF,
      BMP, EXE, ELF, gzip, ... found anywhere inside the file)
    - Entropy: random-looking (encrypted) regions where the format
      shouldn't have them; files that are entirely random data
    - Polyglots: files that are valid as two formats at once
      (e.g. JPEG + ZIP, image + PDF, GIF + JavaScript)
    - Oversized or suspicious metadata (comments, text chunks, tags, ID3
      frames, padding that should be empty, ...)

  PNG, BMP, GIF, TIFF (lossless images)
    - Extraction of the least-significant-bit (LSB) plane
    - Statistical tests: chi-square, sample-pair analysis, RS analysis
    - Tool signatures in the LSB plane: OpenStego header, Stegano-style
      "length:message", length headers, embedded file headers, plain text
    - Extra data hidden inside the compressed PNG image stream or after
      the end of GIF image data

  JPEG
    JPEG hides data in DCT coefficients, not pixels, so pixel-LSB tests are
    NOT used. Instead (modelled on stegdetect by Niels Provos):
    - JSteg, OutGuess 0.13b, JPHide/JPSeek, Invisible Secrets
    - F5: fast mode (leftover encoder comment) and slow mode (statistical)
    - Camouflage / appendX (data appended after the image)
    - Generic "DCT anomaly, tool unknown"

  WAV, FLAC (audio)
    - LSB statistics on the audio samples, LSB activity in silent parts,
      tool / text signatures in the sample LSBs

  MP3
    - Appended data, metadata frames (ID3 etc.), data between audio frames


--------------------------------------------------------------------------------
 6. OPTIONS (Home page)
--------------------------------------------------------------------------------
  Sensitivity (0.1 - 10, default 1.0)
    Multiplies the statistical test results before they are compared with
    the thresholds. Higher finds more but gives more false alarms; lower
    gives fewer false alarms but misses weaker cases.

  False-positive suppression (like stegdetect -n)
    JPEGs that carry a comment field are treated as negative for the DCT
    statistical tests, and the OutGuess test only runs on JPEGs with a
    JFIF version 1.1 header.

  Parallel workers
    How many files are analysed at the same time when scanning a folder.
    Default = number of CPU cores. Lower it to keep the PC more responsive.


--------------------------------------------------------------------------------
 7. EXPORTING REPORTS
--------------------------------------------------------------------------------
  On the Results page click "Export JSON" or "Export CSV".

  JSON  Full report: scan settings, summary counts, and for every file its
        type, status, notes and all findings with their evidence.
  CSV   One row per finding (opens in Excel). Clean and error files get one
        row each, so no file is left out.


--------------------------------------------------------------------------------
 8. COMMAND-LINE VERSION
--------------------------------------------------------------------------------
  stegodetect-cli.exe is the same engine without a window, for batch use:

    stegodetect-cli.exe <file-or-folder> [options]

    -s <number>      sensitivity (default 1.0)
    -n               false-positive suppression
    --json <file>    write a JSON report
    --csv <file>     write a CSV report
    --all            also list clean files in the console

  Example:
    stegodetect-cli.exe "D:\Photos" --json report.json --csv report.csv


--------------------------------------------------------------------------------
 9. LIMITATIONS - PLEASE READ
--------------------------------------------------------------------------------
  - "Clean" means "nothing found", not "guaranteed clean". Good modern
    steganography (for example steghide, which preserves statistics) is
    often not detectable this way.
  - The JPEG tool tests for OutGuess, JPHide and Invisible Secrets, and the
    "DCT anomaly" test, are statistical approximations, not exact copies of
    stegdetect. They were tested on simulated embeddings, not on output
    from the real tools.
  - Low embedding rates may be missed (roughly: OutGuess below ~20-30% of
    capacity, F5 below ~25% in the slow test).
  - SilentEye, steghide and LSB-Steganography leave no fixed marker; they
    can only be caught by the general statistics.
  - Very small JPEGs, very high quality JPEGs and flat / computer-generated
    images give less reliable statistics; such results are limited to LOW
    or skipped (with a note).
  - Not supported: arithmetic-coded, lossless and 12-bit JPEGs (DCT tests
    skipped), MP3Stego-style analysis, and "appended data" checks for
    formats without a clear end marker (gzip, bzip2, xz, ISO, old Office
    .doc/.xls, PSD, BigTIFF, plain text). A note is shown for these.
  - Very large files: pixel/audio statistics use at most 16 million pixels
    or 20 million audio samples; files over 512 MB only get the streaming
    checks (type, appended data, embedded files, entropy).


--------------------------------------------------------------------------------
 10. TROUBLESHOOTING
--------------------------------------------------------------------------------
  "You must install .NET to run this application"
      Install the .NET 9 Desktop Runtime (x64) - see Requirements.

  Windows SmartScreen warns about the app
      The exe is not code-signed. Click "More info" -> "Run anyway" if you
      trust the source.

  Lots of LOW results
      Normal for folders full of icons, logos and other generated graphics.
      Filter them out with the "Low" button on the Results page.

  A file shows ERROR
      It is corrupt, truncated, locked by another program, or you don't
      have permission to read it. The evidence pane shows the reason.

  "Start Scan" is grey
      The path doesn't exist, or a scan is already running.


--------------------------------------------------------------------------------
 11. BUILDING FROM SOURCE
--------------------------------------------------------------------------------
  Needs the .NET 9 SDK.

    git clone <this repository>
    cd StegoDetector
    dotnet build StegoDetector.sln -c Release

  Single-file exe (uses the installed .NET runtime, ~0.6 MB):
    dotnet publish src\StegoDetector.App -p:PublishProfile=SingleFile
    -> publish\StegoDetector.exe

  Self-contained exe (runs without .NET installed, ~60 MB):
    dotnet publish src\StegoDetector.App -c Release -r win-x64
        --self-contained true -p:PublishSingleFile=true
        -p:IncludeNativeLibrariesForSelfExtract=true
        -p:EnableCompressionInSingleFile=true -o publish-standalone

  Test corpus generator (creates clean + simulated stego sample files):
    tools\TestGen\bin\Release\net9.0-windows\TestGen.exe <out-folder> <photo.jpg> [...]


--------------------------------------------------------------------------------
 12. PROJECT STRUCTURE
--------------------------------------------------------------------------------
  src\StegoDetector.Core    Detection engine (no UI)
      Formats\              File type detection, format parsers, decoders
                            (PNG, BMP, GIF, TIFF, JPEG DCT, WAV, FLAC, MP3,
                            ZIP, PDF, PE, ELF, 7z, RAR, ...)
      Checks\Agnostic\      Appended data, carving, entropy, polyglot, metadata
      Checks\Lsb\           Chi-square, sample-pair, RS, LSB signatures
      Checks\Jpeg\          stegdetect-style JPEG tests, calibration
      Engine\               Per-file pipeline, parallel folder scanner
      Reporting\            JSON and CSV export
  src\StegoDetector.App     Windows app (WPF, dark theme)
  src\StegoDetector.Cli     Command-line version
  tools\TestGen             Test sample generator

  Built with C# / .NET 9 / WPF. No third-party libraries.

================================================================================
