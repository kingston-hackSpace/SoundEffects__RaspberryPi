# Sound Effects with RaspberryPi

In the terminal:

```
sudo apt update
sudo apt upgrade -y

mkdir -p ~/Desktop/soundEffects
cd ~/Desktop/soundEffects

sudo apt install python3-pip python3-numpy libportaudio2 -y
sudo apt install python3-venv

python3 -m venv audioenv
source audioenv/bin/activate

pip install sounddevice soundfile numpy

```

-----
Everyday workflow:
Each time you open a terminal:

```
cd ~/Desktop/soundEffects
source venv/bin/activate
python your_script.py
```
When finished:
```
deactivate
```
----

Place your WAV file (name it audio.wav) in your project folder (soundEffects)
Check that your file is in the correct folder:

```
ls
```
----
Create a new Python file:

```
nano play_audio.py
```

The editor opens. Type this code:

```
import soundfile as sf
import sounddevice as sd

# Load WAV file
data, samplerate = sf.read("audio.wav", dtype="float32")

# Play audio
sd.play(data, samplerate)
sd.wait()  # Wait until playback finishes

```
Close your python file by:

Ctrol+O > enter
Control+x

Run your file:

```
python play_audio.py
```

---
Adding sound effects.

Add the following to play_audio.py to understand the audio numbers (raw data numbers). Any effects to apply will consist in manipulating these numbers.

Open your play_audio.py

```
nano play_audio.py
```
add the following:

```
print("Data shape:", data.shape)
print("Data type:", data.dtype)
print("Min value:", data.min(), "Max value:", data.max())
```

----
Add a simple distortion effect:

```
import soundfile as sf
import sounddevice as sd
import numpy as np

# Load WAV
data, samplerate = sf.read("audio.wav", dtype="float32")

print("Data shape:", data.shape)
print("Data type:", data.dtype)
print("Min value:", data.min(), "Max value:", data.max())

# Bitcrush function
def bitcrush(audio, bits=6):
    levels = 2 ** bits
    return np.round(audio * levels) / levels

# Apply effects
crushed_audio = bitcrush(data, bits=4)
gain = 4.0
processed_audio = np.clip(crushed_audio * gain, -1.0, 1.0)

# Play audio
sd.play(processed_audio, samplerate)
sd.wait()
```
