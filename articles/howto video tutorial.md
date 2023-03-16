# How to make a video presentation, lecture or tutorial  using free software.
You may occasionally need to make a video presentation for senior managers or impart your knowledge to colleagues by creating video course/tutorial/lecture.

When I faced with a similar task, I made a research and the approach/solution presented in this guide.
My **hight-level** requirements were:

 1.  Ability to screen & audio record.
 2. Ability to edit video :trim video ,removing stops  and concatenation multiple video clip into one.
 3. Free software should be used. 

In my opinion, the process of creating a good educational video, lecture or presentation is similar to preparing a public presentation, requiring (at least) the following steps:
 1. creating a good story
 2. rehearsing plenty to sound  brief and to the point 

This manual doesn't cover the subject of how to "how to create a killer presentation/lecture/tutorial" .Instead, it focuses on the **technical** aspects of the process. 

## Contents

 1. Video recording.
 2. Video editing.
 3. **Optional**: audio enhancement and video compression.

## Video recording
The most complicated part for me.
Essential to have:
 - story for voice acting
 - recording tool 
 - plenty of record takings

Lets start from **plenty of record takings**:

To be real, it's extremely difficult to record some content the first time without using buzz words, pauses, etc with proper intonation  and pronunciation.

My tips to shorten this activity:

 - Spit and **record** content in small chunks (e.g. it can be easily split into slides for a presentation). Because it is much easier to overwrite a small part than  the entire record.(afterwards small chunks can be concatenated ) 
 - Make a text of your speech (or at least the main points). Reading from a second monitor or a part of the screen is much simpler than doing it from memory(improvisations are not welcome).

Both hints (splitting and reading pre-prepared text) saved me a lot of time.

As for the **recording software**, I recommend using OBS studio, which is the standard for streaming and recording.
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/obs.jpg?raw=true)(https://obsproject.com/)
Of course, if you only want to record a voice-over for a PowerPoint presentation, you can use the built-in recorder in PowerPoint. However, OBS is much more flexible, can be used in a variety of scenarios, and has an intuitive  UI.

OBS UI walkthrough:
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/obs1.jpg?raw=true)

 1. Your video frame, all items are flexible (resizeable , can be placed in necessary order and composition).
 2. **Scene**- list of already created Scenes. 
  A scene is a combination of objects or tools (e. g. video camera capture, audio capture, background media/sound, etc).
 4. **Sources** -components of Scene
 5. Controls to add/remove/scroll Scenes
 6. Controls to add/remove/scroll Sources, related to Scenes
 7. Start recording video
 8. Stop recording video(by default videos saved to C:\Users\ **username** \Videos)

Lets create 2 scenes to cover most use case scenarios:

- **screen capture** to capture demonstrations of some desktop applications with modal pop-up windows.
- **windows capture**  to capture demonstrations of some desktop applications without modal pop-up windows(e,g, PowerPoint).This method is typically much more effective because it records only application screen without interference of Windows Controls or other apps and  allows to open for reading purpose text file while the recording is being made without seeing it on result video.
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/obs2.jpg?raw=true)

To create **screen capture**:

 1. create Scene
 2. Add Sources: Display Capture and Audio Capture
 3. Check the configurations for each source by clicking on it (see screenshot below).For Display Capture be sure that correct monitor selected(in case you have multiple monitors)  and the right audio device is found for Audio Capture.

![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/obs3.jpg?raw=true)
 
 The same steps applied for **window capture** creation; however, *window capture* should be used in place of *display capture* (source section) and in config section window of application ,which you want to record,(e.g. for PowerPoint recording, I recommend to launch presentation/read mode to hide edit controls) should be chosen.

To add your presence, use  **video capture device**.
 
## Video editing

As I mentioned at the beginning, I need the following functionality:
 - Trim video.
 - Concatenate several videos.

So my choice was movie maker, cause of:
 - its free
 - lightweight
 - added to the Windows Store as trusted software**
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/moviemaker.jpg?raw=true)(https://apps.microsoft.com/store/detail/movie-maker-video-editor/9MVFQ4LMZ6C9)

How to use:

 1. Launch movie maker.
 2. Create a new project.
 3. Add  videos  (select **End of Project Mode)**![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/moviemaker1.jpg?raw=true)
 ![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/video%20record%20tutorial/moviemaker2.jpg?raw=true)
 5. Select each clip and click **More tools** button to trim video(if you need it).
 6. To  save video  click **save video** icon ( the bottom of the UI)
 7. Deny saving project(its for payed version).
 8. Save in mp4 format.

## Video enhancement 
To make our video perfect we can:

 - enhance audio quality
 - compress video(lower size- faster load)

Free AI tool from Adobe  can help us with audio enhancement
 (noise removing, making record close to studio quality)
https://podcast.adobe.com/enhance

The mp4 file extension works well with Adobe Enhance despite the fact that mp3 files are required.

Next step in replacement original audio track with enhanced one.
For this purpose I used python's (anaconda distributive) package **moviepy**.

Installation:
```bash
!pip install moviepy
!pip install ffmpeg --upgrade
```
Audio replacement:
```python
import moviepy.editor as mp

f_name='intro'
#Input audio file
audio = mp.AudioFileClip(r'C:\Users\ivolochkov\Downloads\{} (enhanced).wav'.format(f_name))
#Input video file
video = mp.VideoFileClip(r'C:\Users\ivolochkov\Videos\final\new_v\{}.mp4'.format(f_name))
#adding external audio to video
final_video = video.set_audio(audio)
#Extracting final output video
final_video.write_videofile(r"C:\Users\ivolochkov\Videos\final\new_v_en\{}.mp4".format(f_name), fps=30)
```
We have a compressed video file with enhanced audio at the output!
(to tune video output setting **write_videofile** arguments could be modified).


