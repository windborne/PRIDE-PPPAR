# How to Build and Run PRIDE PPP-AR Locally

## 1. Prerequisites

- I am "pretty sure" that on ubuntu a apt install build-essential covers the fortran/gcc requirements.

- `gfortran`, `gcc`, `make` (already used for the supplied binaries)
- `python3` with `matplotlib`/`numpy` if you want to use the plotting scripts

Everything runs locally; no external installation or system-wide `make install` is required.

## 2. Build / Rebuild the executables

You can either trust the install script (I did), simply do a chomod to make the install.sh exicutable and run it. 

OR

```bash
cd /~/src/PRIDE-PPPAR/src
make
```

This populates the solver binaries under the various `src/*/` directories when they are not yet compiled or when changes were made (e.g., to `src/spp`). But I think you have to manually go copy pdp3 to /PRIDE-PPPAR/pdd3 and set it to exicutable.

## 3. Prepare a working PATH

`pdp3` calls the compiled binaries (`spp`, `lsq`, `redig`, `tedit`, etc.) by name. Add the build directories plus `scripts/` to your PATH before running:

```bash
cd ~/src/PRIDE-PPPAR
export PATH="$(pwd)/src/arsig:$(pwd)/src/lsq:$(pwd)/src/tedit:$(pwd)/src/spp:$(pwd)/src/redig:$(pwd)/src/orbit:$(pwd)/src/utils:$(pwd)/src/otl:$(pwd)/src/mhm:$(pwd)/scripts:$(pwd):$PATH"
```

If you reopen a shell, re-run the `export` (or place it in your shell init file while you are working in this repo).

## 4. Configuration

`config/local.cfg` points PRIDE to the bundled tables and to the local product cache (`products/`). You can edit an alternate copy if you want different settings, but the shipped file is ready for the sample flights.

## 5. Running PPP-AR on the sample flights

From the repository root:

```bash
# Flight 1 (auto-detects site name "balo")
pdp3 -cfg config/local.cfg  -n balo data/flight1.obs 

# Flight 2 (MARKER NAME is blank, so provide a 4-char ID)
pdp3 -cfg config/local.cfg -n FLT2 data/flight2.obs
```

So here is the deal - PRIDE will FAIL if you dont have a 4 character ID in the header for the rinex file. That's what the -n flag does.

Each run creates a session directory under `2025/YYYY-DOY[/…]`. Key outputs per site include:

- `kin_*` – kinematic trajectory -- BREAD CRUMB TRAIL OF GPS POINTS (NEEDED IN FURTHER PROCEESSING)
- `stt_*` – statistics report
- `res_*` – residuals
- `rck_*` – receiver clock series -- RECEIVER CLOCK FILE (NEEDED IN FURTHER PROCEESSING)
- `log_*` – edit log
- `ztd_*`, `amb_*`, `stt_*`, etc., plus supporting precise products cached under `products/common/`

If you rerun a flight, any existing directory with the same date is overwritten.

## 6. Plotting (optional) UNTEESTED

The plotting scripts use Python and/or GMT. Example calls once the PATH is set:

```bash
plottrack.py 2025/171-172/kin_2025171_balo balo_track
plotres.py   2025/171-172/res_2025171_balo G12
plotztd.py   2025/171-172/ztd_2025171_balo balo_ztd
```

These generate PNGs in the current directory. GMT-based `*.sh` scripts expect GMT to be installed.

## 7. Processing other data

1. Place your RINEX observation file in `PRIDE-PPPAR/YYYY/data/` (or point to any accessible path).
    - So PRIDE has a little bug where if your GPS RINEX file goes over a GPS day boundry (like 171-172) it requires the data to be saved under a directory in the YYYY directory (Where YYYY is 2025, 2026, etc).
2. Ensure the required broadcast ephemeris (`brdm*.p`) is present or let `pdp3` download it.
    - When you run install.sh it will ask if you want to use the international GNSS data products - as long as your machine can access the internet this should grab them from there and cache them locally.
3. Run `pdp3 -cfg config/local.cfg yourfile.obs` (add `-n SITE` if the file lacks a four-character marker name). I already made a note about this. 

All precise products are cached under `products/` for reuse; remove that directory if you want a clean re-download.

