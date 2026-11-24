# Roadmind

Traffic lights that think about people.

We are team IIUC_ZeroCode from International Islamic University Chittagong, and we made Roadmind for the CoU AI Hackathon 2026.

## The idea

At most junctions in our cities, the signal either runs on a fixed timer or doesn't work at all, so the traffic police control it by hand. A bus full of people can sit at a red light while the green goes to a road with two cars. The timer doesn't know the difference.

So we thought, what if the signal looked at how many people are waiting, not how many vehicles?

Roadmind uses the camera that is already at the junction. Our AI finds the vehicles and people on each road, estimates how many people are waiting, and tells the traffic officer in Bangla which road should get more green time. The officer presses Accept or Override. Accept sends the new timing to the lights. Override lets the officer set his own time and pick a reason. If the officer does nothing, the lights just keep running on their normal timer like they do now.

## How we count people

The AI can't see inside a bus, and for privacy we don't want it to. So we use the average number of people each vehicle usually carries:

- bus 40, leguna 12, battery rickshaw 4, CNG 3, rickshaw 2
- car 1.5, motorbike 1.3, truck 1.2, bicycle 1

These are averages over many vehicles, so 10 cars count as about 15 people. The numbers are shown in the app under Assumptions and the officer can change them.

Example: one bus on Road 1 (about 40 people) and ten motorbikes on Road 2 (about 13 people). If you count vehicles, Road 2 wins. If you count people, Road 1 wins. Roadmind gives the green to Road 1.

## What's inside

- YOLO11n (Ultralytics) to detect cars, buses, trucks, motorbikes, bicycles and people
- ByteTrack to follow each vehicle and know how long it has been waiting
- Python, FastAPI and OpenCV for the backend, plain HTML/CSS/JS for the dashboard
- Gemini (optional) to make the Bangla sentence sound natural. The numbers always come from our own calculation, and if Gemini changes a number we don't use its answer
- edge-tts (optional) so the Speak button reads the advice in Bangla

Some rules we added:

- every road gets at least 10 and at most 60 seconds of green
- if vehicles on a road have waited more than 2 minutes, that road goes first
- yellow is always 3 seconds, Roadmind never touches it
- if 8 or more people are waiting to cross, they get extra walking time
- if the camera is dark, blurry or blocked, it doesn't give advice and says to keep the normal timer
- it finds the roads by itself by watching the traffic for about 20 seconds
- every decision is saved in a log
- the Emergency button gives an ambulance's road green first (this part is only a simulation for now)
- **Ask Roadmind** (chat box): the officer can ask anything about the junction in Bangla, Banglish or English, like a normal chat. For example "a mela crowd is coming on Road 2, it will double, give me a plan for 1 hour", "after office time Road 1 and 3 get busy, 45 minutes", or "Road 2 will get empty". Any road, several roads, any length of time, a crowd going up or down. Roadmind answers in Bangla with a green time for each road, how many people would be stuck with the normal timer and with the plan, and says when extra police are needed. With a Gemini key, Gemini reads the question and writes the reply like a chat assistant, but every number comes from the camera and our rules: if Gemini writes a number we didn't give it, we don't show that reply. Without internet, Roadmind's own rules read the question and answer
- **Heavy rain**: in rain a bus or rickshaw can hide behind a truck, so the count can be too low. When it rains (the officer presses 🌧, or the picture is blurry) and a vehicle is partly behind a truck, Roadmind doesn't cut any road's green time

No number plates are read and no faces are identified. The video stays on the computer.

## How to run it

You need Python 3.11 or newer. We used 3.14 on Windows.

First time:

```
py -3.14 -m venv .venv
.venv\Scripts\activate
python -m pip install -r requirements.txt
copy .env.example .env
```

Every time:

```
.venv\Scripts\activate
python -m uvicorn app.main:app
```

Then open http://localhost:8000. The first run downloads the YOLO model, after that it works without internet. If you want Gemini, put your key in `.env` after `GEMINI_API_KEY=`.

To try your own footage, press "Upload video" at the top, or press "Camera" and type a phone's IP Webcam address, a CCTV RTSP link, or `0` for the laptop webcam. Use a camera that is high up and sees traffic from at least two directions.

Problems we ran into on Windows:

- `pip.exe` blocked by Application Control: use `python -m pip` instead
- error about Visual C++ Redistributable or `c10.dll`: install https://aka.ms/vs/17/release/vc_redist.x64.exe
- "running scripts is disabled": run `Set-ExecutionPolicy -Scope Process Bypass` first

## Running it online

There is a Dockerfile, so it runs on Render or any Docker host. Our junction video (`data/clips/demo.mp4`) is in the repo, so the online version starts by itself, nothing to upload. On Render's free plan set `ROADMIND_SLOW_MODE=1` and `ROADMIND_IMG_SIZE=320`.

### Why the online version is slower than on our laptop

- **The free server is small.** Render's free plan gives a tiny part of one CPU and no graphics card. Our laptop has many CPU cores, so the AI checks 10 to 20 pictures a second there. Online it checks about 1 picture a second.
- **The video still plays smoothly.** We play the video at normal speed and the AI works on the newest picture it can get. So the boxes around the vehicles move a little late and jump instead of sliding. The counts and the advice are still correct, because traffic doesn't change much in one second.
- **A lighter stream.** Online we send about 12 pictures a second, a bit smaller and a bit more compressed, so it also loads well on slow internet.
- **Finding the roads takes longer.** For a new uploaded video the video waits for the AI while the roads are being found (the AI has to see the cars move). After that it plays at normal speed. Our own junction video already has its roads saved, so it starts smoothly.
- **The first visit is slow.** A free Render app goes to sleep when nobody uses it. The first visit can take up to a minute while it wakes up and loads the AI model.

For the full speed, run it on a laptop (see "How to run it").

## What's not done yet

- the model doesn't know CNG, rickshaw or leguna yet, we need to train it on Bangladeshi vehicles
- people per vehicle is an estimate, not a real count
- sometimes a lorry is detected as a bus
- we haven't tested it at a real junction yet, so we don't know how much waiting time it saves. The traffic light panel in the app stands in for a real signal controller
- we haven't measured the accuracy on a big dataset yet. `scripts/evaluate_counts.py` can compare the AI's count with a hand count

## Credits

All libraries, models and fonts with their licences are in [CREDITS.md](CREDITS.md). We used Claude to help with code and documents, Gemini for the Bangla sentences in the app, and YOLO for detection. Since YOLO is AGPL-3.0, this project is open source under the same licence.

## Team

- Ehsanul Alam
- K.A Shahriar Rubaiyat
- MD. Ikram Hossain Sami
