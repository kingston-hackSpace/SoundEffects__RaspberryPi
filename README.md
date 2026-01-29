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

Looping the noise:

```
import soundfile as sf
import sounddevice as sd
import numpy as np

# -------------------------------
# LOAD AUDIO
# -------------------------------
data, sr = sf.read("audio.wav", dtype="float32")
if data.ndim == 1:
    data = data[:, np.newaxis]  # make stereo-compatible

# -------------------------------
# PARAMETERS
# -------------------------------
chunk_size = 4096            # frames per callback
noise_amount_glitch = 0.01   # light glitch effect
frames_per_5s = sr * 5       # 5-second interval

# Precompute glitch noise for the whole track
noise_glitch = np.random.randn(len(data), data.shape[1]) * noise_amount_glitch

# -------------------------------
# CALLBACK FUNCTION
# -------------------------------
start_frame = 0  # global frame counter

def callback(outdata, frames, time, status):
    global start_frame
    if status:
        print(status)  # comment out after debugging

    end_frame = start_frame + frames
    chunk = data[start_frame:end_frame]

    # Wrap around if we reach the end
    if len(chunk) < frames:
        chunk = np.pad(chunk, ((0, frames - len(chunk)), (0, 0)), mode='wrap')

    # Apply glitch every 5 seconds
    if (start_frame // frames_per_5s) % 2 == 1:
        chunk += noise_glitch[start_frame:end_frame]

    # Clip to valid range
    outdata[:] = np.clip(chunk, -1.0, 1.0)

    start_frame += frames

# -------------------------------
# START STREAM
# -------------------------------
with sd.OutputStream(
    samplerate=sr,
    channels=data.shape[1],
    blocksize=chunk_size,
    callback=callback,
):
    print("Playing audio with 5-second glitch intervals. Ctrl+C to stop.")
    while True:
        pass



```
