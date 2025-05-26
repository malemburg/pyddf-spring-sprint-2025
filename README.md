# Python Meeting Düsseldorf - Spring Sprint 2025
- May 24-25 2025
- At Eviden / Atos offices in Düsseldorf, Seestern

## Links
- Python Meeting Düsseldorf: https://www.pyddf.de/
- Sprint Meetup page: https://www.meetup.com/python-meeting-dusseldorf/events/307026678/

# Intro

## Topics

- Marc-André: Mobile App Development with Flet
- Jens: Mobile App Development with BeeWare
- Philipp: Cloud providers using OSS only and non-US
- Jochen: MCP – connecting LLMs to APIs
- Charlie: Work on openpyxl code generator
- Sven: TikZ support for Python, using matplotlib
- Andy: Interested in documentation generators
- Klaus: Work on FritzConnection 2.0 (only Sunday)

# My topic: Application Development with Flet

- Participants:
	- Marc-André: https://github.com/malemburg
	- Jens: https://github.com/jedie/

## Resources
- Flet
    - [Homepage: Build multi-platform apps in Python powered by Flutter | Flet](https://flet.dev/)
    - [Roadmap | Flet](https://flet.dev/roadmap/)
- Sprint
    - Github project: https://github.com/malemburg/pyddf-spring-sprint-2025
    - Github project: https://github.com/jedie/pyddf-flet-test

## Notes

From the homepage:

> Flet enables developers to easily build realtime web, mobile and desktop apps in Python. No frontend experience required.

### Webview approach
- Flet seems to generally use the concept of a webview, where the app is running in a server app, which then uses a websocket to connect to the frontend for rendering

- The Flet Android app uses the same approach
	- [Testing Flet app on Android | Flet](https://flet.dev/docs/getting-started/testing-on-android/)
	- It connects to a server running locally on your machine
	- Displays the UI and provides events back to the server
	- The app itself is not running on the phone


### Build environment
- When trying to build a proper APK, `flet build` downloads SDKs:
	- **WARNING:** This will take a looong time and use lots of RAM (up to 16GB)
	- [Publishing Flet app to multiple platforms | Flet](https://flet.dev/docs/publish/)
	- [Packaging app for Android | Flet](https://flet.dev/docs/publish/android/)
	- Flutter SDK
		- ?? env var
	- Android SDK
		- set ANDROID_HOME to the dir where the SDK is installed
	- Chrome
		- set CHROME_EXECUTABLE to a Chrome compatible browser
		- needed for web app runs: `felt run --web`
	- Visual Studio 2022
		- needed to build Windows executables
		- not automatically downloaded
	- Wheels are available to provide extra functionality using native code
		- [Built-in binary Python packages for Android and iOS | Flet](https://flet.dev/docs/reference/binary-packages-android-ios/)

### Mobile app permissions and sensor access
- Mobile app permissions and extra sensor packages can be added in the build process
- Extra controls
	- https://flet.dev/docs/publish/#including-optional-controls
- Permissions
	- https://flet.dev/docs/publish/#permissions
	- for Android: https://flet.dev/docs/publish/android/#permissions

### Building APKs (Android)
- these are needed for mobile apps on Android
- use verbose mode: `flet build -v apk`
	- this provides more details and helps find issues
- **external dependencies don't appear to work in Flet 0.28.3**
	- the build command does not seem to include these or does not make the app in the APK aware of the included external deps
	- hard to debug, since the APK files only contain the app compiled as a `libapp.so` file
- to reduce the APK size, it is possible to have `flet build` create separate APKs per architecture:
	- `flet build apk --split-per-abi`
		- this will produce three files, one for arm64-v8a (ARM64), armeabi-v7a (ARM) and x86_64 (Intel x64)
		- most phones today need *arm64-v8a*
		- the Android emulator needs *x86_64*
	- additionally, it is possible to build only for specific architectures using: `flet build apk --arch arm64-v8a x86_64` (space separated)
		- this can cause problems with the emulator, though, with the app not starting

### Creating a Flet app
- see https://flet.dev/docs/getting-started/create-flet-app/
- start with `flet create`
- this will create a project dir layout
- code lives in `src/`
- the main entry point is `src/main.py`
	- this will have to run `ft.app(main)`, with `main()` being a function taking a `page` parameter and adding UI controls to this page
	- `page` is the main application window
- Flet implements imperative UI model where you "manually" build application UI by adding them to a page
	- *aside:* Flutter uses a declarative approach

### Adding controls
- see https://flet.dev/docs/getting-started/flet-controls/
- simply append the controls to the `page` object using `page.add(control)`
- then run `page.update()` to have these rendered
- controls added to a page (or other control) are typically stored in `.controls`
- controls have a `.page` attribute which allows accessing the page they were added to
- each control gets a `.uid` assigned to uniquely identify the control in the DOM (assignement happens when the control is added to the page)
- controls can be made invisible using `control.visible = False`
	- this is handy when quickly replacing parts of the layout with new content
	- you add all components to the page and then selectively make the ones you want to work with visible
- controls can be accessed using the list interface
- by using layout controls, they can be stacked, nested, etc.
- to clear a page (the window), use `page.clean()`

### Event handlers
- see https://flet.dev/docs/getting-started/flet-controls/#event-handlers
- some controls have events, e.g. buttons have an `on_click` parameter which deifnes the event handler to call when it is pressed
- event handlers get called with an event parameter

### Setting focus on controls
- to give e.g. an entry field focus, use `control.focus()`

### Control values
- values, such as the text shown by a text control, can typically be accessed via `.value`

### Custom controls
- see https://flet.dev/docs/getting-started/custom-controls/
- it is easily possible to create new controls by subclassing existing ones
- for composite controls, use the `.controls` list  to manage controls part of the layout control
- handlers which can be overridden during a control's lifecycle: https://flet.dev/docs/getting-started/custom-controls/#life-cycle-methods
- updates to the controls can be handled explicitly by the control, by calling `self.update()` when necessary (isolated mode), or have this done implicitly on all children (non-isolated modee)
	- see https://flet.dev/docs/getting-started/custom-controls/#isolated-controls on how to declare a control isolated

### UI looks can be adapted
- see https://flet.dev/docs/getting-started/adaptive-apps/
- by setting `page.adaptive = True`, Flet will automatically switch between Material design controls and Cupertino ones, depending on the platform
- `page.platform` allows querying the current platform, when designing your own controls
- `page.web` allows querying whether the app is run in a browser or not

### Routing
- see https://flet.dev/docs/getting-started/navigation-and-routing/
- Flet supports routing to show different parts of the app
- the current route is available as `page.route`
- there's an event handler which can be set to get infos about route changes: `page.on_route_change`
- it's possible to go to a different route by
	- assigning to `page.route`
	- or by calling `gae.go(new_route)`, which then also calls the change handler and the update method
- different routes can be mapped to *views* in Flet
	- views are stacked on top of a page
	- they are available via the `page.views` list
	- the back button triggeres a `on_view_pop` event handler
		- if can be used to update `page.views` and navigate back to e.g. the root
- parameters on routes can be matched using `TemplateRoute`
	- see https://flet.dev/docs/getting-started/navigation-and-routing/#route-templates for details

### Flet Flutter packages
- see https://flet.dev/docs/publish/#including-optional-controls
- some functionality needs both a Python and a Flutter lib
- these have to be added separately, not via standard deps

### Better way to have auto-reload
- the standard mechanism in `flet run` does not seem to work in the current version 0.28.3; the app hangs after a reload
- but you can use watchdog directly:
	- `watchmedo auto-restart --pattern="*.py" --recursive -- flet run`

### Async
- see https://flet.dev/docs/getting-started/async-apps/
- Flet can run event handlers async
	- simply declare the event handler as an async function and then set this up as e.g. an on_click handler
		- beware that the handlers may not block !!!
	- the whole app can be run async using `await ft.app_async(main)`
	- *note:* sync even handlers are run in threads by Flet
- To run background tasks, you can use `page.run_task()`
	- this needs an async function to run
	- https://flet.dev/docs/getting-started/async-apps/#threading

## Jens' notes
- https://github.com/jedie/pyddf-flet-test

## Conclusion
- Flet looks *very* promising, but it'll probably take a few more months to get to a state where you can use it in production

# Other things to check out

Some new tools / services I learned about during the sprint.

##  Local Send
- app to send data between devices
- [LocalSend: Share files to nearby devices](https://localsend.org/)

## OpenRouter
- SaaS which provides a unified access to many different LLMs
	- https://openrouter.ai/
- supports routing by pricing and throughput
- provides a unified API to many LLMs and centralized payments
- also provides statistics on latency and tps (tokens per second)

## Chutes
- LLM inference provider
	- https://chutes.ai/app
- has many models available for free
- Python interface:
	- https://pypi.org/project/chutes/
	- https://github.com/rayonlabs/chutes

# Other sprint projects

These are some notes I took in the summary sessions. The notes are not necessarily complete or represent the true results of what was achieved.

## MCP with a database
- Philipp

Idea: Connect Claude to a database on Azure

- https://github.com/punkpeye/awesome-mcp-servers
- https://github.com/runekaagaard/mcp-alchemy
	- MCP server to access databases using SQLAlchemy
- connected to Azure MS SQL database

Results
- LLM showed relationship analysis
- found company name in data
	- bummer: privacy alert /!\
- restarted after removing permissions to tables which had the name in them
- issue: context windows used up quickly if the database is larger
- but: goal accomplished - it's possible to extract interesting facts from the database without having to know SQL

Better approach for development:
- create mostly empty database
- then let LLM generate some test data
- figured out foreign keys without seeing them (Philipp didn't add them)
- reran analysis
- provides good results

Tried an MCP server for Azure
- ran some queries against this

Tried to integrate MCP servers into MS Copilot
- didn't work due to problems with getting MS Copilot to work

Checked out IONOS Stackable infrastructure
- very basic, some Kubernestes only
- they provide basic building blocks, but no integrations
- couldn't test much due to credit card issues
- Terraform supports the platform

Checked out OVH Cloud
- no CC issues
- runs near Calais
- comes with
	- data catalog
	- lakehouse manager
		- includes dependencies on Snowflake ?!
	- data processing engine (Trino ?)
	- analytics manager
- company behind this: ForePaaS
	- got acquired by OVH
- Terraform supports the platform
- too opinioiated

## MCP with a Django codebase
- Jochen

- few shot templating to generate plugins for his resume system
	- includes CSS, Python examples and corresponding prompts
	- LLM generates files which get run by the Django app
- goal: have a MCP server which provides the needed data
	- not ready yet
	- needs a standardized API for code interogation and introspection
- did achieve something in this direction
	- hard to connect the Claude desktop LLM to the local MCP servers
	- he also tried using a Playwright MCP server
		- quickly reaches the token limit
		- doesn't go very far
	- he used Claude to write most of the code

## openpyxl code generation
- Charlie
- Andy

- older project
	- build code to resemble a given schema
	- works, but the generated code doesn't look nice

## TikZ
- Sven

- matplotlib interface with output using TikZ
- slowly recreating the various graphic elements
- added classes for more sturcture
- added a dispatcher
- a bit difficult, since matplotlib uses different concepts for doing the graphics
- still: good start

## FritzConnection
- Klaus

- preparing version 2.0
	- some parts need rewrites, due to new features in more modern FritzBoxes
- spent most of the time learning argparse
	- uses subcommands
