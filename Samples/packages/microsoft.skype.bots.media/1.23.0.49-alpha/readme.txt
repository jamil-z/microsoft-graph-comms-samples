Microsoft Real-Time Media Platform for Bots API preview

Microsoft.Skype.Bots.Media API reference documentation is available at:
https://microsoftgraph.github.io/microsoft-graph-comms-samples/docs/bot_media/index.html.

For an overview of the Real-Time Media Platform for Bots, please see:
https://docs.microsoft.com/en-us/microsoftteams/platform/bots/calls-and-meetings/real-time-media-concepts

See https://github.com/microsoftgraph/microsoft-graph-comms-samples
for sample code and to report issues or ask questions.

Recent Changes and New Features in the Microsoft.Skype.Bots.Media API
=====================================================================

1.23:

  !! IMPORTANT !!
  This update takes a dependency on Microsoft.Extensions.Logging 
  and Microsoft.Extensions.Logging.Abstractions version >= 2.1.1.
  If nuget restore does not automatically generate binding redirects, 
  binding redirects would need to be added manually for the missing DLLs.
  
  For example see below. `X.0.0.0` represents the version number of the Assemblies 
  that got binplaced to the output directory during the nuget restore/build

  <dependentAssembly>
  <assemblyIdentity name="Microsoft.Extensions.Logging" publicKeyToken="adb9793829ddae60" culture="neutral" />
  <bindingRedirect oldVersion="0.0.0.0-X.0.0.0" newVersion="X.0.0.0" />
  </dependentAssembly>
  <dependentAssembly>
    <assemblyIdentity name="Microsoft.Extensions.Logging.Abstractions" publicKeyToken="adb9793829ddae60" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-X.0.0.0" newVersion="X.0.0.0" />
  </dependentAssembly>

* Bug fixes:
  - fixed a bottleneck within the SDK's media engine that could cause
  severe delay setting up new media sessions containing VideoSockets.
  In some cases, the delay could result in meeting join failures for
  the bot.

  - fixed a bug which could cause the VideoSocket.Send API to throw an
  exception when sending video frames using a "raw" format such as NV12.

  Thank you to our partners for reporting bugs and issues to us!

1.22:

* New AudioSocket.GetSendBuffersPendingCount API reports how many audio
  buffers are currently queued in the AudioSocket to be sent. Audio is
  always played out at a constant rate of one frame every 20ms. If the
  bot sends audio too quickly (more than one frame every 20ms), the
  buffers will queue up and this can be an indication that audio may be
  "falling behind" video frame playout. A bot which sends audio from a
  real-time source should occasionally query this API to monitor that
  audio buffers are not queueing up too much. A return value of 50, for
  example, means that one second's worth of audio buffers is currently
  queued to be sent. The AudioSocket.Send API will throw an exception
  if the number of queued audio buffers exceeds 100.

* New MediaPlatform.CurrentVideoSocketCount API reports how many VideoSocket
  objects the application has currently instantiated. There is a limit of
  1000 simultaneous VideoSocket objects which may be instantiated at any given
  time.

* Fixed support for sending portrait (9x16) aspect ratio video.
  In order to send video with a portrait aspect ratio, all the formats in the
  VideoSocketSettings.SupportedSendVideoFormats list must have a 9x16 ratio
  resolution (e.g., 180x320, 720x1280, 1080x1920).

* Bug fixes:
  - fixed: some receivers might observe frozen or blank images viewing the
  video from a bot sending encoded H.264 video frames.

1.21: (minor update, no release notes)

1.20:

* Removed support for .NET 4.6.2. Now supports .NET Framework 4.7.2 only.

* New AudioMediaBuffer JitterBufferLength and RoundTripDelay properties.

* Bug fixes:
  - fixed: AudioVideoFramePlayer did not resume playing if the Audio/VideoSocket
  MediaSendStatus changed to Inactive and then back to Active.

1.19:

* Bug fixes:
  - fixed: A bug introduced in 1.18 where a null reference exception was 
  being thrown when the bot is run either attached to a debugger, 
  or in the Azure Cloud Service emulator.

  Thank you to our partners for reporting bugs and issues to us!

1.18:

* AudioSocket.ToneReceived event args expose a new property, SequenceId.
  The AudioSocket should raise tones in order, but the sequence ID can be 
  used to detect missing tones or if the bot's tone processing is asynchronous.

* Bug fixes:

  - fixed: an undocumented limit of 50 simultaneous VideoSockets could
  be configured to receive video in the "raw"/decoded NV12 color format.
  If the bot tried to create more than 50 concurrent VideoSockets to
  receive NV12-format video, many VideoSocket.Subscribe API operations
  would fail silently and the VideoSocket would not receive any video
  frames. The fix increases the limit to 1000 simultaneous VideoSockets.

  - fixed: a small memory leak per bot call.

  Thank you to our partners for reporting bugs and issues to us!

1.17:

* AudioSocket/VideoSocket.GetQualityOfExperienceData() methods expose 
  additional quality metrics.

* Bug fixes:

  - fixed: A media platform memory leak could occur due to a race condition
  during resolution changes in the video encoded (H264) send path. The leak 
  could also have led to crashes under certain conditions.

  - fixed: The first video frame received in the video receive path would
  have 0 as the MediaSourceId instead of the ID of the source.

  Thank you to our partners for reporting bugs and issues to us!

1.16:

* IMPORTANT! The media libraries now require the Visual C++ 2019
  Runtime for x64. You can obtain the VC++ x64 runtime redistributable
  (VC_redist.x64.exe) from:

  https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads

  As part of the startup of the real-time media bot's VM, the
  VC_redist.x64.exe must be executed in order to ensure the runtime is
  installed. The following is a sample script to do this:

  @echo off
  set logfile=C:\InstallCppRuntime-%MONITORING_ROLE%.log
  Startup\VC_redist.x64.exe /quiet
  echo %date% %time% ErrorLevel=%errorlevel% >> %logfile%
  exit /b %errorlevel%

* Bug fixes:

  - fixed: A media platform crash could occur when a bot participates
  in a large meeting and sends video in a "raw" format, such as NV12.

  - fixed: the VideoSendStatusChangedArgs.MediaType property would
  always indicate MediaType.Audio instead of the VideoSocket's actual
  MediaType value. The VideoMediaStreamQualityChangedEventArgs class
  was also missing a MediaType property.

  Thank you to our partners for reporting bugs and issues to us!

1.15:

* AudioSocket/VideoSocket.GetQualityOfExperienceData() methods
  provide media quality metrics such as packet loss and jitter.
  The bot may query these QoE metrics periodically to monitor the
  health of its media streams.

1.14:

* AudioSocketSettings.ReceiveUnmixedMeetingAudio option allows the bot
  to receive separate, unmixed audio of the active speakers in a meeting.
  The AudioMediaBuffer.UnmixedAudioBuffers property provides the unmixed
  audio frames of the active speakers.

  Note: unmixed audio is optimized for machine cognition (e.g., speech
  recognition/transcription) rather than for human listening. Certain
  error concealment treatments (e.g, to mitigate packet losss) are not
  applied to received audio in unmixed mode.

* AudioSocket.SendDtmfTone/SendDtmfTones() methods allow the bot to
  send DTMF tones. These APIs might be useful in testing an IVR-like
  system.
