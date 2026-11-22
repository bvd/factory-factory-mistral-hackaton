# Getting started

> **22 November 2026:** the hackathon server has been removed, so the address and the account page below no longer work. The page is kept as a record: the same steps work on any Debian server you can reach over ssh.

Three steps take you from nothing to talking with your own agent on the server:

1. [Connect to the server](#1-connect-to-the-server)
2. [Start your Vibe session](#2-start-your-vibe-session)
3. [Speak to your Vibe session](#3-speak-to-your-vibe-session)

Each step ends with a check, so you know where you stand before you go on. What was tried beforehand, and what was not, is listed [at the end](#what-was-tried).

## 1. Connect to the server

You need an ssh key and a role code from the organiser.

1. **Make a key**, if you have none:

   ```
   ssh-keygen -t ed25519
   ```

   This makes two files in the folder `.ssh` in your home folder. `id_ed25519.pub` is the public key: that is the one you hand in. `id_ed25519` is the private key: it never leaves your laptop.

2. **Get your account.** Open https://app.hierax.eu, log in with your Google account and redeem the role code. Under "Hackathon server", choose a user name (3 to 16 characters: lower-case letters, digits and hyphens, starting with a letter), paste the content of `id_ed25519.pub` and press "Create my account with this key".

   No Google account? Send the organiser your public key and the user name you want.

3. **Save the address**, so that `ssh hackathon` is enough from now on. Add this to the file `config` in that same `.ssh` folder, with your own user name. Make the file if it is not there; on Windows it is `C:\Users\YOU\.ssh\config`, without `.txt`.

   ```
   Host hackathon
     HostName app.hierax.eu
     Port 22180
     User YOUR-NAME
   ```

   The rest of this page uses `ssh hackathon`. Without the shortcut it is `ssh -p 22180 YOUR-NAME@app.hierax.eu`.

4. **Log in:**

   ```
   ssh hackathon
   ```

   The first time, ssh shows the fingerprint of the server's key and asks whether to go on. Say yes only if it shows this one:

   ```
   SHA256:AYDvvRoj3uB792ZYzjiN2Fo4nUoQtMzIjLW/nV85AS0
   ```

**Check.** `whoami` shows your user name, `groups` shows `team`, and `sudo -n true` gives no error.

What you are on:

- Debian 12 with 8 cores, 32 GB memory and 300 GB disk. Installed: git, curl and tmux. Nothing else.
- You are an administrator of the machine (sudo without a password), and so is every teammate. Anyone on the team can therefore read any file on it, also in your home folder. Put nothing there that the team may not see.
- `/srv/team` is the folder the team shares.
- The server reaches the internet and nothing else. It is not backed up, and it is removed after the hackathon: what is not pushed is gone.
- When your role code runs out, your account is blocked and your files stay. A new code from the organiser, and handing in your key again, opens it again.

## 2. Start your Vibe session

Do this on the server, in your own account.

1. **Install Vibe:**

   ```
   curl -LsSf https://mistral.ai/vibe/install.sh | bash
   ```

   This is Mistral's own installer. It puts `uv`, a recent Python and Vibe in your home folder and needs no sudo. The server's own Python is 3.11 and Vibe needs 3.12 or later, so the installer brings one along.

   The installer may end with an error that `vibe` is not on your PATH. That is expected here: the folder it installs into did not exist yet when you logged in. Log out and in again (`exit`, then `ssh hackathon`).

2. **Open a tmux session named `vibe`:**

   ```
   tmux new -A -s vibe
   ```

   Inside tmux your session keeps running when your connection drops, and the same command brings you back to it. Ctrl+B and then D leaves it running and returns you to the plain shell. The bar at the bottom of the terminal starts with the name of the session you are in: `[vibe]`. That name is used again in step 3.

3. **Start Vibe in a work folder:**

   ```
   mkdir -p ~/work && cd ~/work && vibe
   ```

4. **Sign in.** Vibe asks for a theme and then how to sign in. On this server, choose **Use an API key**: make a key for the hackathon at https://chat.mistral.ai/code/extensions (Code, Vibe CLI) and paste it. The same key works with a free account and with a paid plan. Vibe saves it in `~/.vibe/.env`.

   The other choice, **Launch browser**, also works without a browser on the server: press `c` to copy the sign-in address, open it in the browser on your laptop and sign in there.

   Either way a credential of yours is then on a server where every teammate is an administrator. That is why a key made for the hackathon is the better choice: delete it in your Mistral account when the hackathon is over.

**Check.** `/whoami` shows your account and your plan. Then ask something that needs the machine, such as: *Which machine am I on? Run `uname -a` and tell me.* Vibe asks for your approval before it runs the command.

Useful to know: `/help` lists all commands, Shift+Tab switches how much Vibe may do without asking, `vibe --continue` picks up your last conversation, and `vibe --setup` runs the sign-in again.

## 3. Speak to your Vibe session

Vibe has a voice mode (`/voice`, then Ctrl+T), but it records from the machine Vibe runs on. A server has no microphone, and the Linux package of Vibe (3.1.4) is built without recording: it answers "Voice capture is not available in this build". So the sound has to come from your laptop:

```
microphone > ffmpeg > ssh > "ears" on the server > Mistral realtime transcription > text, typed into your Vibe session
```

Your laptop only sends sound. A Mistral model on the server side turns the speech into text, so rule 2 holds and your Mistral key stays in one place.

Sending the sound is one command, and it works. The program on the server, called `ears` here, does not exist yet: building it is the first job you give your Vibe session.

### 3.1 On your laptop: ffmpeg and the name of your microphone

Install ffmpeg: `winget install Gyan.FFmpeg` on Windows, `brew install ffmpeg` on macOS, `sudo apt install ffmpeg` on Debian or Ubuntu.

| Laptop | Find the microphone | Write it in the commands below as |
|---|---|---|
| Windows | `ffmpeg -hide_banner -list_devices true -f dshow -i dummy` and take the line that ends in `(audio)` | `-f dshow -i audio="NAME"` |
| macOS | `ffmpeg -hide_banner -f avfoundation -list_devices true -i ""` and take the number in the list of audio devices | `-f avfoundation -i ":0"` |
| Linux | nothing to find | `-f pulse -i default` |

On Windows, two things matter:

- Run the commands in **Command Prompt** (`cmd`), not in PowerShell. Windows PowerShell changes the sound on its way through the pipe, and what arrives is noise.
- If the name of your microphone has a special character in it, such as ®, use the "Alternative name" that ffmpeg prints on the line below it.

### 3.2 Check the route with five seconds of sound

Record five seconds on your laptop and store them on the server. The Windows form is shown; on macOS or Linux, replace `-f dshow -i audio="NAME"` as in the table.

```
ffmpeg -loglevel error -f dshow -i audio="NAME" -t 5 -ac 1 -ar 16000 -f s16le - | ssh hackathon "cat > mic-test.pcm"
```

Play them back from the server:

```
ssh hackathon "cat mic-test.pcm" | ffplay -loglevel error -autoexit -nodisp -f s16le -ar 16000 -ch_layout mono -
```

An older ffmpeg may want `-ac 1` in place of `-ch_layout mono`.

**Check.** You hear yourself, and `ssh hackathon "ls -l mic-test.pcm"` shows about 160 000 bytes: 32 000 for each second.

### 3.3 Let Vibe build its ears

Give your Vibe session this job. Change it as you like: it is a starting point, not a design.

```
Write a program ~/bin/ears in Python.

It reads raw sound from standard input: 16-bit signed little-endian PCM, mono, 16000 Hz.
It streams that sound to Mistral's realtime transcription, model voxtral-mini-transcribe-realtime-3,
as described on https://docs.mistral.ai/studio/audio/speech-to-text/transcribe-in-realtime
(the Python package is "mistralai[realtime]").

Each time I have finished a sentence, it types that sentence into a tmux session with
"tmux send-keys -t SESSION -l" and then sends Enter. The session is "vibe" unless I pass
--session NAME. With --no-enter it leaves the text in the input box for me to send, with a
space after each sentence.

It prints what it hears to standard error, so that I can follow it on my laptop.

It takes the API key from MISTRAL_API_KEY, or else from ~/.vibe/.env.

It is started over ssh without a login shell ("ssh hackathon bin/ears"), so it must not
rely on PATH: uv is in ~/.local/bin. Make the file executable.

Test it with the recording ~/mic-test.pcm, typing into a tmux session named "ears-test"
that you make for the test, and not into "vibe". Make that session detached, do not switch
my terminal to it, and remove it when the test is done. Show me what arrived there.
```

If you signed in through the browser, there is no key in `~/.vibe/.env` yet. Make one as in step 2 and put it in that file as `MISTRAL_API_KEY=your_key`.

### 3.4 Talk

`ears` types into whatever is in front in the tmux session `vibe`. So look at your terminal on the server first: the bar at the bottom starts with `[vibe]`, and Vibe is on screen. If a plain shell is there instead, your words are run as commands.

Then, on your laptop, in a second window:

```
ffmpeg -loglevel error -f dshow -i audio="NAME" -ac 1 -ar 16000 -f s16le - | ssh hackathon bin/ears --no-enter
```

With `--no-enter` what you say waits in Vibe's input box and you press Enter yourself. Begin that way. Without it, every finished sentence is sent at once.

Ctrl+C stops it.

- The window on your laptop shows what `ears` hears. If text shows there and nothing arrives in Vibe, check on the server that a session named `vibe` exists: `tmux ls`.
- While it runs, everything your microphone picks up goes to the server and on to Mistral, including the people next to you. Stop it when you are not dictating.
- What you say is typed into Vibe as if you had typed it. Vibe still asks before it runs a command, unless you switched that off with Shift+Tab. Leave it on while you speak.
- If the text lags behind on Windows, put `-audio_buffer_size 50` before `-i`.

**Check.** You say "list the files in this folder", the sentence appears in Vibe's input box, you press Enter, and Vibe answers.

**Lost in tmux?** `tmux ls` lists the sessions. Ctrl+B and then S lets you choose one. `tmux kill-session -t NAME` removes one, with whatever runs in it. If Vibe ended up outside the session `vibe`, the simplest way back is `tmux kill-server`, which removes all sessions, followed by `tmux new -A -s vibe` and `cd ~/work && vibe --continue`, which picks up your last conversation.

## What was tried

On 21 November 2026, from a Windows laptop with ffmpeg 8.1:

| | |
|---|---|
| **Works** | Step 1: an account through the Google login and the "Hackathon server" card, an account made by hand with a key, the login on port 22180 with the fingerprint above, and sudo. Step 2: the installer (Vibe 3.1.4) and the sign-in with an API key. Step 3.2: five seconds of speech from a laptop microphone arrived on the server. A test tone arrived unchanged from Command Prompt and from Git Bash; Windows PowerShell 5.1 did change it. Step 3.3: a Vibe session built `ears` from the prompt above. Fed with that recording, also at the speed of speech and without a login shell, it typed the right sentence into a test session, with the key from `~/.vibe/.env`. |
| **Read, not run** | That the Linux package has no recording: found by looking inside the package of version 3.1.4. |
| **Not tried** | Step 3.4: speaking live into a running Vibe session. The sign-in through the browser. The commands for macOS and Linux. |

If a step fails for you, say so in an issue on this repository, with the command and what it answered.
