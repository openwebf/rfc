CSS & Style support V2
=============

# Motivation

Currently, mostly implementations of CSS & Style are written in Dart and running in Flutter ui thread. 

With the release of isolate thread support in WebF, the JavaScript running now had been moving to a isolate thread that running parallaly alone side with the Flutter ui thread. 

The Style and CSS process takens siginificate times from CSS codes to computable styles for each renderObjects before the layout begins. 

The long running in Flutter ui thread could cause jank for the UI and user would complains the bad performance for apps that built using Flutter and WebF. Most of the profile data shows that some janks was lead to the time comsumed by the style stage, which the application is trying to parse CSS stylesheets and calculate the target styles for each of elements, and it's impossible to finish this jobs in less than 1 frame (16ms) and cause losing frame and jank. 

For those of reasons, it's time to move to the next architecture version of CSS & Style support in WebF, and leverage the advantages of dedicated thread that the long running of CSS parsing and style computing wouldn't affect the state of Flutter ui thread.

The another part of CSS & Style V2 support is to support more CSS selectors and pseudos and compatiable to web browsers. It's unadvisble to spend tons of times to re-implement the CSS support that web browser had done perfect and completely. 

For the next architecture of CSS & Style support of WebF, we will based on the source codes of [Blink's CSS Support](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/core/css), not only these C++ codes will be running in the JavaScript thread, but also save our efforts to keep compatibility with web browsers and licenses free. 


## ToDo

1. Style & Script Loading Phases
2. Animation & KeyFrames
3. Widgets System
4. Load Events
5. SVG

Attributes 