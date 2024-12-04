### Notes on WebViews
- Im not really sure how to test these right now. A lot of AVD images have WebView versions that don't work very well. WebViews also just suck.
- Freed doesn't work on most AVDs
### MobSF: magical GUI tool for dummies
#### They maintain docker images, which is nice
```
$ docker pull opensecurity/mobile-security-framework-mobsf:latest

$ docker run -it --rm -p 8000:8000 -e MOBSF_ANALYZER_IDENTIFIER="emulator-5554" opensecurity/mobile-security-framework-mobsf:latest
```

#### Start your AVD with this script
```
#!/bin/bash

# Check if START_PORT, AVD_NAME, and open_gapps.zip are provided as arguments
AVD_NAME=$1
START_PORT=${2:-5554}
GAPPS_ZIP=$3

# Determine the emulator path based on OS
if [ "$(uname)" = "Darwin" ]; then
  EMULATOR_PATH="$HOME/Library/Android/sdk/emulator/emulator"
elif [ "$(uname)" = "Linux" ]; then
  EMULATOR_PATH="$HOME/Android/Sdk/emulator/emulator"
else
  echo "$(tput setaf 1)Unsupported OS: $(uname)$(tput sgr0)"
  exit 1
fi

# Check if the emulator path exists and is executable
if [ ! -x "$EMULATOR_PATH" ]; then
  echo "$(tput setaf 1)Emulator not found at $EMULATOR_PATH$(tput sgr0)"
  exit 1
fi

# Locate adb path
if [ -x "$HOME/Android/Sdk/platform-tools/adb" ]; then
  ADB_PATH="$HOME/Android/Sdk/platform-tools/adb"
elif [ -x "$HOME/Library/Android/sdk/platform-tools/adb" ]; then
  ADB_PATH="$HOME/Library/Android/sdk/platform-tools/adb"
elif command -v adb >/dev/null 2>&1; then
  ADB_PATH=$(command -v adb)
else
  echo "$(tput setaf 3)adb not found in common locations. Defaulting to 'adb' in PATH.$(tput sgr0)"
  ADB_PATH="adb"
fi

# Check if adb is available
if [ ! -x "$ADB_PATH" ]; then
  echo "$(tput setaf 1)adb not found at $ADB_PATH$(tput sgr0)"
  exit 1
fi

# If no AVD_NAME is provided, list available AVDs and exit
if [ -z "$AVD_NAME" ]; then
  echo -e "$(tput bold)Available AVDs:$(tput sgr0)\n"
  "$EMULATOR_PATH" -list-avds
  echo -e "$(tput bold)\nUse any Android AVD 5.0 - 11.0, up to API 30 without Google Play (production image).$(tput sgr0)"
  echo "$(tput bold)Usage: $0 AVD_NAME [START_PORT] [open_gapps.zip path]$(tput sgr0)"
  echo "$(tput bold)Example: $0 Pixel_6_Pro_API_28 5554 /path/to/open_gapps.zip$(tput sgr0)"
  exit 1
fi

# Check if provided AVD_NAME exists in available AVDs
AVAILABLE_AVDS=$("$EMULATOR_PATH" -list-avds)
if ! echo "$AVAILABLE_AVDS" | grep -q "^$AVD_NAME$"; then
  echo "$(tput setaf 1)Error: AVD $AVD_NAME not found in available AVDs.$(tput sgr0)"
  echo "$(tput bold)Available AVDs:$(tput sgr0)"
  echo "$AVAILABLE_AVDS"
  exit 1
fi

# Kill any existing emulator processes
for pid in $(pgrep -f emulator); do
  echo "Killing emulator process with PID $pid"
  kill -9 "$pid"
done

# Start the emulator with user-defined AVD and port
"$EMULATOR_PATH" -avd "$AVD_NAME" -writable-system -no-snapshot -wipe-data -port "$START_PORT" >/dev/null 2>&1 &
echo "$(tput setaf 2)Starting AVD $AVD_NAME on port $START_PORT$(tput sgr0)"
echo "Waiting for emulator to boot..."
"$ADB_PATH" wait-for-device

# Check if socat is available only on Linux
if [ "$(uname)" = "Linux" ]; then
  if command -v socat >/dev/null; then
    SOCAT_AVAILABLE=true
  else
    SOCAT_AVAILABLE=false
    echo "$(tput setaf 3)Warning: socat is not installed. Skipping port forwarding with socat. This might be required for docker in Linux.$(tput sgr0)"
  fi
else
  SOCAT_AVAILABLE=false
fi

# Increment ports and start socat listeners if socat is available
if [ "$SOCAT_AVAILABLE" = true ]; then
  LISTEN_PORT=$((START_PORT + 2))
  TARGET_PORT=$((START_PORT + 1))
  # Remove all previous forwards
  for pid in $(lsof -t -i TCP:$LISTEN_PORT -sTCP:LISTEN -c socat); do
    echo "Killing socat process on port $LISTEN_PORT with PID $pid"
    kill -9 "$pid"
  done
  # Start a single socat listener
  socat TCP-LISTEN:$LISTEN_PORT,fork,reuseaddr TCP:localhost:$TARGET_PORT &
  
  echo "socat listener started on port $LISTEN_PORT forwarding to $TARGET_PORT in the host."
  echo "$(tput bold)Docker users please set the environment variable MOBSF_ANALYZER_IDENTIFIER=host.docker.internal:$LISTEN_PORT for adb connectivity.$(tput sgr0)"
fi

# Install Play Store if open_gapps.zip is provided
if [ -n "$GAPPS_ZIP" ]; then
  if [ ! -f "$GAPPS_ZIP" ]; then
    echo "$(tput setaf 1)Error: File $GAPPS_ZIP not found.$(tput sgr0)"
    exit 1
  fi

  echo "Installing PlayStore"

  if ! command -v unzip >/dev/null; then
    echo "$(tput setaf 1)Error: unzip is not installed.$(tput sgr0)"
    exit 1
  fi

  if ! command -v lzip >/dev/null; then
    echo "$(tput setaf 1)Error: lzip is not installed.$(tput sgr0)"
    exit 1
  fi
  
  PLAY_EXTRACT_DIR="play"
  [ -d "$PLAY_EXTRACT_DIR" ] && rm -rf $PLAY_EXTRACT_DIR
  unzip "$GAPPS_ZIP" 'Core/*' -d ./$PLAY_EXTRACT_DIR || { echo "$(tput setaf 1)Error: Unzip failed$(tput sgr0)"; exit 1; }
  rm $PLAY_EXTRACT_DIR/Core/setup*
  lzip -d $PLAY_EXTRACT_DIR/Core/*.lz || { echo "$(tput setaf 1)Error: Decompression failed$(tput sgr0)"; exit 1; }
  for f in $(ls $PLAY_EXTRACT_DIR/Core/*.tar); do
    tar -x --strip-components 2 -f $f -C $PLAY_EXTRACT_DIR || { echo "$(tput setaf 1)Error: Extraction failed$(tput sgr0)"; exit 1; }
  done

  echo "Waiting for the emulator to be ready..."
  sleep 25
  echo "Installing PlayStore components"
  "$ADB_PATH" root
  "$ADB_PATH" remount >/dev/null 2>&1
  "$ADB_PATH" push $PLAY_EXTRACT_DIR/etc /system
  "$ADB_PATH" push $PLAY_EXTRACT_DIR/framework /system
  "$ADB_PATH" push $PLAY_EXTRACT_DIR/app /system
  "$ADB_PATH" push $PLAY_EXTRACT_DIR/priv-app /system
  "$ADB_PATH" shell stop
  "$ADB_PATH" shell start
  [ -d "$PLAY_EXTRACT_DIR" ] && rm -rf $PLAY_EXTRACT_DIR
  echo "PlayStore installed successfully"
else
  echo "$(tput setaf 3)No open_gapps.zip provided. Skipping Play Store installation.$(tput sgr0)"
fi
```

##### Like this
```
$ ./start_avd.sh

Available AVDs:

Medium_Phone_API_35
Pixel_5_API_30
Pixel_6a_API_29

$ ./start_avd.sh Pixel_6a_API_29
```

#### Notes on MobSF
- Dynamic analysis features have pretty strict requirements:
	- AVD Image must be API 30 (Android R, Android 11) or below 
	- AVD Image must be non-production (doesn't come with Google Play installed)
- But this means that any app that uses Play services will likely just crash on launch
- BUT: you can supply Google Play dependencies statically by downloading them from https://opengapps.org and running the AVD like this: `./start_avd.sh Small_Phone_API_28 5554 open_gapps-arm64-9.0-stock-20220215.zip`

### Intercepting Proxies
This is all pretty self-explanatory given your experience fuzzing web apps with burp suite. You can do pretty much the same thing here. Just install burp suite, configure the AVD proxy, and install the burp cert in the AVD.

The interesting part is defeating cert pinning, which otherwise prevents the use of intercepting proxies. We can do that by patching the application!
### Patching Applications
##### Installing Frida and objection
```
critt@Mac workbench % python3 -m venv frida 
critt@Mac workbench % source frida/bin/activate
(frida) critt@Mac workbench % pip3 install frida-tools
(frida) critt@Mac workbench % pip3 install objection
(frida) critt@Mac workbench % pip3 install --force-reinstall -U setuptools
(frida) critt@Mac workbench % objection

Usage: objection [OPTIONS] COMMAND [ARGS]...

  

       _   _         _   _

   ___| |_|_|___ ___| |_|_|___ ___

  | . | . | | -_|  _|  _| | . |   |

  |___|___| |___|___|_| |_|___|_|_|

        |___|(object)inject(ion)

       Runtime Mobile Exploration

          by: @leonjza from @sensepost

  

  By default, communications will happen over USB, unless the --network option

  is provided.
```

##### Injecting Frida manually
https://frida.re/docs/gadget/
https://koz.io/using-frida-on-android-without-root/
- The idea here is to inject Frida in the form of a shared library (.so)
- Roughly speaking:
	1. Obtain Frida .so file and put it in the libs folder
	2. Inject `System.loadLibrary("<frida_lib_name>")` call in launch/main Activity
		- Note: this is smali, not java
	3. Rebuild + resign application
	4. Install
	5. Control Frida via objection while application is running
- Its essentially just rebuilding the application from sources after modifying it to include something akin to trojan, with a bunch of malicious features

#### Frida + objection
```
critt@Mac workbench % objection explore
critt@Mac workbench % android sslpinning disable
```
