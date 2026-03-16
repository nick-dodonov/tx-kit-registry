# Private Registry Update Integrity Tool

Tool for updating SHA hashes in `source.json` files of a private Bazel registry.

## Description

This tool is similar to `update_integrity` from the Bazel Central Registry (BCR), but is designed for working with private registries. It:

1. Downloads the source archive from the URL in `source.json`
2. Calculates the new SHA hash of the archive
3. Updates the hashes of all files in the `overlay/` directory
4. Updates the hashes of all files in the `patches/` directory (if present)
5. Saves the updated `source.json`

## Usage

### Python script (direct)

```bash
cd /path/to/your/registry
python3 tools/update_integrity.py <module_name> [--version <version>]

# Examples:
python3 tools/update_integrity.py lwlog
python3 tools/update_integrity.py lwlog --version 1.4.0
```

### Shell wrapper (recommended)

```bash
cd /path/to/your/registry
./tools/update_integrity.sh <module_name> [version]

# Examples:
./tools/update_integrity.sh lwlog
./tools/update_integrity.sh lwlog 1.4.0
```

## Options

- `module` - module name to update (required)
- `--version` / second argument - module version (optional, latest by default)
- `--registry` - path to registry root (current directory by default)

## When to Use

Run this tool when:

1. **Before committing changes** to overlay files (BUILD.bazel, MODULE.bazel, etc.)
2. **After adding new patch files**
3. **When changing the source URL** in source.json
4. **To verify consistency** after changes in the registry

## Example Workflow

```bash
# 1. Make changes to overlay files
echo 'cc_library(name = "test")' >> modules/lwlog/1.4.0/overlay/BUILD.bazel

# 2. Update integrity hashes
./tools/update_integrity.sh lwlog 1.4.0

# 3. Commit changes
git add .
git commit -m "Update lwlog overlay"
git push
```

## Registry Structure

The tool works with registries of the following structure:

```
registry_root/
├── bazel_registry.json          # Registry configuration
├── modules/                     # Modules directory (specified in bazel_registry.json)
│   └── lwlog/                   # Module name
│       └── 1.4.0/              # Module version
│           ├── source.json      # Source metadata (updated by tool)
│           ├── MODULE.bazel     # Bazel module
│           ├── overlay/         # Overlay files (BUILD.bazel, MODULE.bazel, etc.)
│           │   ├── BUILD.bazel
│           │   └── MODULE.bazel
│           └── patches/         # Patch files (optional)
│               └── fix.patch
└── tools/                       # Registry tools
    ├── update_integrity.py      # Python script
    └── update_integrity.sh      # Shell wrapper
```

## source.json Format

```json
{
    "url": "https://github.com/user/repo/archive/refs/tags/v1.4.0.tar.gz",
    "integrity": "sha256-gQChBE/WKk6+r2DaCOrSfe8O3rkmBv2IvIb2btVNu0A=",
    "strip_prefix": "repo-1.4.0",
    "overlay": {
        "BUILD.bazel": "sha256-pdROS+Fn0sloYo+pFFIqhKmRRqMOEQivJjnMIZltw0w=",
        "MODULE.bazel": "sha256-ktr0L1PKsQ/bg7lwCxBerIXpnr2toud02tqrgzPuZtA="
    },
    "patches": {
        "fix.patch": "sha256-AdCdGcITmkauv7V3eA0SPXOW6XIBvH6tIQouv/gjne4="
    }
}
```

## Dependencies

- Python 3.6+
- Python standard library (json, hashlib, urllib, pathlib)
- Internet access for downloading archives

## Comparison with BCR

| Feature | BCR update_integrity | This tool |
|---------|---------------------|-----------|
| Target | Bazel Central Registry | Private registries |
| Dependencies | click, special BCR structure | Python standard library only |
| Configuration | Hardcoded for BCR | Reads bazel_registry.json |
| Modules | All BCR modules | Your registry modules |

## Troubleshooting

### Error "Registry root does not exist"
Make sure you run the tool from the registry root or use the correct path in `--registry`.

### Error "Module not found"
Check the module name:
```bash
ls modules/  # View available modules
```

### Error "Failed to download"
Check the URL in source.json and internet connectivity.

### Error "source.json not found"
Make sure the `source.json` file exists for the specified module version.