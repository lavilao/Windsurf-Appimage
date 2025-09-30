# Windsurf AppImage Builder

This repository builds AppImage packages for the Windsurf AI code editor automatically using GitHub Actions.

## What is Windsurf?

Windsurf is an AI-first code editor that combines the power of artificial intelligence with traditional code editing features. It provides intelligent code completion, generation, and assistance to enhance developer productivity.

## What is an AppImage?

AppImage is a format for distributing portable software on Linux without needing superuser permissions to install the application. It's a single executable file that contains the application and all its dependencies.

## Features

- ✅ Automatically builds AppImage from latest Windsurf releases
- ✅ No installation required - just download and run
- ✅ Works on most Linux distributions
- ✅ Includes proper desktop integration
- ✅ SHA256 checksums for integrity verification
- ✅ Automated releases via GitHub Actions

## Download

### Latest Release

You can download the latest AppImage from the [Releases](../../releases) page.

### Direct Download

The workflow automatically creates releases when new versions are detected. Look for releases tagged with `latest-{version}`.

## Installation & Usage

1. **Download the AppImage:**
   ```bash
   wget https://github.com/yourusername/Windsurf-Appimage/releases/download/latest-x.x.x/Windsurf-x.x.x-x86_64.AppImage
   ```

2. **Make it executable:**
   ```bash
   chmod +x Windsurf-*.AppImage
   ```

3. **Run Windsurf:**
   ```bash
   ./Windsurf-*.AppImage
   ```

### Desktop Integration (Optional)

To integrate with your desktop environment:

```bash
# Move to a permanent location
mkdir -p ~/.local/bin
mv Windsurf-*.AppImage ~/.local/bin/windsurf.AppImage

# Create desktop entry
mkdir -p ~/.local/share/applications
cat > ~/.local/share/applications/windsurf.desktop << 'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=Windsurf
GenericName=AI Code Editor
Comment=The AI-first code editor
Exec=/home/$USER/.local/bin/windsurf.AppImage %F
Icon=windsurf
Categories=TextEditor;Development;IDE;
StartupWMClass=windsurf
StartupNotify=true
Keywords=windsurf;code;editor;ide;development;ai;
MimeType=text/plain;inode/directory;application/x-code-workspace;
Actions=new-empty-window;

[Desktop Action new-empty-window]
Name=New Empty Window
Exec=/home/$USER/.local/bin/windsurf.AppImage --new-window %F
Icon=windsurf
EOF
```

## Build Process

The AppImage is built automatically using GitHub Actions whenever:

- Code is pushed to the main branch
- A new release is published
- The workflow is manually triggered

### Build Steps

1. **Fetch Source:** Downloads the latest PKGBUILD from AUR to get version and source URL
2. **Download:** Downloads the official Windsurf .deb package
3. **Extract:** Extracts the .deb package contents
4. **Package:** Creates an AppDir structure with proper AppRun script
5. **Build:** Uses appimagetool to create the final AppImage
6. **Test:** Performs basic functionality tests
7. **Release:** Uploads the AppImage and checksums to GitHub releases

### Manual Build

If you want to build locally:

```bash
# Clone this repository
git clone https://github.com/yourusername/Windsurf-Appimage.git
cd Windsurf-Appimage

# Install dependencies (Ubuntu/Debian)
sudo apt-get install wget curl file

# Fetch PKGBUILD
curl -o PKGBUILD https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=windsurf-bin

# Source the PKGBUILD to get variables
source ./PKGBUILD

# Download the .deb file
wget "${source[0]}" -O windsurf.deb

# Extract
dpkg-deb -x windsurf.deb windsurf-extracted

# Create AppDir
mkdir -p windsurf.AppDir/usr
cp -r windsurf-extracted/usr/* windsurf.AppDir/usr/
cp app.desktop windsurf.AppDir/windsurf.desktop

# Create AppRun
cat > windsurf.AppDir/AppRun << 'EOF'
#!/bin/bash
HERE="$(dirname "$(readlink -f "${0}")")"
export PATH="${HERE}/usr/bin:${PATH}"
export LD_LIBRARY_PATH="${HERE}/usr/lib:${HERE}/usr/lib/x86_64-linux-gnu:${LD_LIBRARY_PATH}"
exec "${HERE}/usr/bin/windsurf" "$@"
EOF
chmod +x windsurf.AppDir/AppRun

# Download appimagetool
wget https://github.com/AppImage/appimagetool/releases/download/continuous/appimagetool-x86_64.AppImage
chmod +x appimagetool-x86_64.AppImage

# Build AppImage
export ARCH=x86_64
./appimagetool-x86_64.AppImage windsurf.AppDir Windsurf-${pkgver}-x86_64.AppImage
```

## Troubleshooting

### AppImage won't run

1. **Check permissions:**
   ```bash
   ls -la Windsurf-*.AppImage
   ```
   If not executable, run: `chmod +x Windsurf-*.AppImage`

2. **Check FUSE:**
   ```bash
   # Install FUSE if missing
   sudo apt-get install fuse libfuse2  # Ubuntu/Debian
   sudo yum install fuse fuse-libs     # CentOS/RHEL
   sudo pacman -S fuse2                # Arch Linux
   ```

3. **Extract and run manually:**
   ```bash
   ./Windsurf-*.AppImage --appimage-extract
   ./squashfs-root/AppRun
   ```

### Desktop integration issues

- Ensure `~/.local/share/applications` is in your desktop environment's search path
- Try logging out and back in
- Run `update-desktop-database ~/.local/share/applications`

## Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Test the build process
5. Submit a pull request

## License

This repository contains build scripts and configuration files. The Windsurf editor itself is proprietary software by Codeium. Please refer to Windsurf's official license terms.

## Disclaimer

This is an unofficial AppImage build. For official support, please contact Codeium directly.

## Related Links

- [Windsurf Official Website](https://codeium.com/windsurf)
- [AppImage Documentation](https://appimage.org/)
- [AUR Windsurf Package](https://aur.archlinux.org/packages/windsurf-bin)