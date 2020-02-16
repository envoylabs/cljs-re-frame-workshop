# Re-Frame workshop

NB: This repo is available at the short URL `https://tinyurl.com/ll-feb-20-workshop`

The goal of this workshop is to better understand how re-frame event flows work, and how to build and debug re-frame apps.

The basics of the framework are simple, and scale pretty well, so we'll be going over the basics of the entire re-frame event flow.

## Getting started

First, [clone the repo here](). 

Then, run the shadow-cljs watcher for the first time. Shadow is a build tool that bridges CLJ and Node, allowing for more seamless interop between the two platforms.

    $ npx shadow-cljs watch app

The first time, it will take a while to run.

When the build has finished, it will log the ports it is running on, and then something like:

    [:app] Build completed. (512 files, 0 compiled, 0 warnings, 25.08s)

When that happens, you can navigate to port `8080` in your browser and open the app.

There's also a shorthand in the NPM runner:

    $ npm run dev:watch

## Task 1 - Fix subs

In order to get started, check out the `workshop-task-1` branch. You'll find that things are now broken. Good news - we're going to fix them!

Open `subs.cljs` in `src/cljs/todomvc` and let's have a look.

You'll see lots of documentation - that's because this is a canonical re-frame example and is used as living docs in the main re-frame repo.

You'll notice that the `:showing` sub no longer works. It's referenced in the view code with:

```clj
(subscribe [:showing])
```

The sub is relatively simple - it's just extracting the value of the top-level `:showing` key from the db.

We're going to implement it on line 15.

```clj
(reg-sub
  :showing
  (fn [db _]
    (:showing db)))
```

## Task 2 - Fix events

For the next task, we're going to check out the `workshop-task-2` branch. Again, you'll find things are broken. Again, we're coming to the rescue!

Open `events.cljs` in `src/cljs/todomvc` and let's have a look.

## Task 3 - New feature

Okay, this is a somewhat contrived feature, but we're going to add a character counter.

Every time the textbox value changes, we'll update a counter in the app-db, then reveal that value on screen.

We'll use an event called `:update-text-edits`, which will track the number of total changes to the textbox.

Steps:

- Implement an event, `:update-text-edits`.
- Make a place for the data in the `app-db` in `db.cljs`
- Implement a sub, `:total-text-edits`
- Subscribe to the sub and show the data in the view

In some cases, using events like this for something that changes at a high frequency could result in big performance issues, particularly on mobile.

For more information, looking at Reagent (the React templating library used under the hood) and form-3 components is useful.

You can also debounce effects. This would be achieved by something like:

```clj
(ns todomvc.debounce
  (:require [re-frame.core :refer [reg-fx dispatch]]
            [schema.core :as s])) ;; this ns uses schema to validate inputs etc

(defn now [] (.getTime (js/Date.)))

(def registered-keys (atom nil))

(def DebouncedEventSchema
  {:key s/Keyword
   :event [s/Any]
   :delay s/Num})

(defn dispatch-if-not-superceded [{:keys [key delay event time-received]}]
  (when (= time-received (get @registered-keys key))
    ;; no new events on this key!
    (dispatch event)))

(defn dispatch-later [{:keys [delay] :as debounce}]
  (js/setTimeout
   (fn [] (dispatch-if-not-superceded debounce))
   delay))

;; works in a similar fashion to the lodash debounce fn
(reg-fx
 :dispatch-debounce
 (fn dispatch-debounce [debounce]
   (try
     (s/validate DebouncedEventSchema debounce)
     (catch js/Object e
       (error e)))
   (let [ts (now)]
     (swap! registered-keys assoc (:key debounce) ts)
     (dispatch-later (assoc debounce :time-received ts)))))
```

You could then use it like so:

```clj
;; now you call this event instead of the original one
(reg-event-fx
 :debounced-update-text-edits
 (fn [fx [_ text]]
   {:dispatch-debounce {:key :update-text-edits ;; unique key
                        :event [:update-text-edits text] ;; the original event
                        :delay 250}}))
```

## REPLs and `s/emacs/$yr-editor/i`

Your best bet is to read the shadow-cljs docs [here](https://shadow-cljs.github.io/docs/UsersGuide.html#cider). The CLJS REPL is a lot more usable than it was even a year or two ago.

## Adding tests

Some scaffolding for tests has been added via Karma. You'll need to install it via NPM to get cracking though.

    $ npm install -g karma-cli

Then you will be able to run:

    $ lein karma

___

**NB: these are the default Re-Frame docs. I've left them in because they may be useful as extra reading**

### Project Overview

* Architecture:
[Single Page Application (SPA)](https://en.wikipedia.org/wiki/Single-page_application)
* Languages
  - Front end ([re-frame](https://github.com/day8/re-frame)): [ClojureScript](https://clojurescript.org/) (CLJS)
* Dependencies
  - UI framework: [re-frame](https://github.com/day8/re-frame)
  ([docs](https://github.com/day8/re-frame/blob/master/docs/README.md),
  [FAQs](https://github.com/day8/re-frame/blob/master/docs/FAQs/README.md)) ->
  [Reagent](https://github.com/reagent-project/reagent) ->
  [React](https://github.com/facebook/react)
* Build tools
  - Project task & dependency management: [Leiningen](https://github.com/technomancy/leiningen)
  - CLJS compilation, REPL, & hot reload: [`shadow-cljs`](https://github.com/thheller/shadow-cljs)
  - Test framework: [cljs.test](https://clojurescript.org/tools/testing)
  - Test runner: [Karma](https://github.com/karma-runner/karma)
* Development tools
  - Debugging: [CLJS DevTools](https://github.com/binaryage/cljs-devtools),
  [`re-frame-10x`](https://github.com/day8/re-frame-10x)

#### Directory structure

* [`/`](/../../): project config files
* [`dev/`](dev/): source files compiled only with the [dev](#running-the-app) profile
  - [`cljs/user.cljs`](dev/cljs/user.cljs): symbols for use during development in the
[ClojureScript REPL](#connecting-to-the-browser-repl-from-a-terminal)
* [`resources/public/`](resources/public/): SPA root directory;
[dev](#running-the-app) / [prod](#production) profile depends on the most recent build
  - [`index.html`](resources/public/index.html): SPA home page
    - Dynamic SPA content rendered in the following `div`:
        ```html
        <div id="app"></div>
        ```
    - Customizable; add headers, footers, links to other scripts and styles, etc.
  - Generated directories and files
    - Created on build with either the [dev](#running-the-app) or [prod](#production) profile
    - Deleted on `lein clean` (run by all `lein` aliases before building)
    - `js/compiled/`: compiled CLJS (`shadow-cljs`)
      - Not tracked in source control; see [`.gitignore`](.gitignore)
* [`src/cljs/todomvc/`](src/cljs/todomvc/): SPA source files (ClojureScript,
[re-frame](https://github.com/Day8/re-frame))
  - [`core.cljs`](src/cljs/todomvc/core.cljs): contains the SPA entry point, `init`
* [`test/cljs/todomvc/`](test/cljs/todomvc/): test files (ClojureScript,
[cljs.test](https://clojurescript.org/tools/testing))
  - Only namespaces ending in `-test` (files `*_test.cljs`) are compiled and sent to the test runner

### Editor/IDE

Use your preferred editor or IDE that supports Clojure/ClojureScript development. See
[Clojure tools](https://clojure.org/community/resources#_clojure_tools) for some popular options.

### Environment Setup

1. Install [JDK 8 or later](https://openjdk.java.net/install/) (Java Development Kit)
2. Install [Leiningen](https://leiningen.org/#install) (Clojure/ClojureScript project task &
dependency management)
3. Install [Node.js](https://nodejs.org/) (JavaScript runtime environment)
4. Install [karma-cli](https://www.npmjs.com/package/karma-cli) (test runner):
    ```sh
    npm install -g karma-cli
    ```
5. Install [Chrome](https://www.google.com/chrome/) or
[Chromium](https://www.chromium.org/getting-involved/download-chromium) version 59 or later
(headless test environment)
    * For Chromium, set the `CHROME_BIN` environment variable in your shell to the command that
    launches Chromium. For example, in Ubuntu, add the following line to your `.bashrc`:
        ```bash
        export CHROME_BIN=chromium-browser
       ```
7. Clone this repo and open a terminal in the `todomvc` project root directory
8. Download project dependencies:
    ```sh
    lein deps && npm install
    ```

### Browser Setup

Browser caching should be disabled when developer tools are open to prevent interference with
[`shadow-cljs`](https://github.com/thheller/shadow-cljs) hot reloading.

Custom formatters must be enabled in the browser before
[CLJS DevTools](https://github.com/binaryage/cljs-devtools) can display ClojureScript data in the
console in a more readable way.

#### Chrome/Chromium

1. Open [DevTools](https://developers.google.com/web/tools/chrome-devtools/) (Linux/Windows: `F12`
or `Ctrl-Shift-I`; macOS: `⌘-Option-I`)
2. Open DevTools Settings (Linux/Windows: `?` or `F1`; macOS: `?` or `Fn+F1`)
3. Select `Preferences` in the navigation menu on the left, if it is not already selected
4. Under the `Network` heading, enable the `Disable cache (while DevTools is open)` option
5. Under the `Console` heading, enable the `Enable custom formatters` option

#### Firefox

1. Open [Developer Tools](https://developer.mozilla.org/en-US/docs/Tools) (Linux/Windows: `F12` or
`Ctrl-Shift-I`; macOS: `⌘-Option-I`)
2. Open [Developer Tools Settings](https://developer.mozilla.org/en-US/docs/Tools/Settings)
(Linux/macOS/Windows: `F1`)
3. Under the `Advanced settings` heading, enable the `Disable HTTP Cache (when toolbox is open)`
option

Unfortunately, Firefox does not yet support custom formatters in their devtools. For updates, follow
the enhancement request in their bug tracker:
[1262914 - Add support for Custom Formatters in devtools](https://bugzilla.mozilla.org/show_bug.cgi?id=1262914).

## Development

### Running the App

Start a temporary local web server, build the app with the `dev` profile, and serve the app with
hot reload:

```sh
lein dev
```

Please be patient; it may take over 20 seconds to see any output, and over 40 seconds to complete.

When `[:app] Build completed` appears in the output, browse to
[http://localhost:8280/](http://localhost:8280/).

[`shadow-cljs`](https://github.com/thheller/shadow-cljs) will automatically push ClojureScript code
changes to your browser on save. To prevent a few common issues, see
[Hot Reload in ClojureScript: Things to avoid](https://code.thheller.com/blog/shadow-cljs/2019/08/25/hot-reload-in-clojurescript.html#things-to-avoid).

Opening the app in your browser starts a
[ClojureScript browser REPL](https://clojurescript.org/reference/repl#using-the-browser-as-an-evaluation-environment),
to which you may now connect.

#### Connecting to the browser REPL from your editor

See
[Shadow CLJS User's Guide: Editor Integration](https://shadow-cljs.github.io/docs/UsersGuide.html#_editor_integration).
Note that `lein dev` runs `shadow-cljs watch` for you, and that this project's running build id is
`app`, or the keyword `:app` in a Clojure context.

Alternatively, search the web for info on connecting to a `shadow-cljs` ClojureScript browser REPL
from your editor and configuration.

For example, in Vim / Neovim with `fireplace.vim`
1. Open a `.cljs` file in the project to activate `fireplace.vim`
2. In normal mode, execute the `Piggieback` command with this project's running build id, `:app`:
    ```vim
    :Piggieback :app
    ```

#### Connecting to the browser REPL from a terminal

1. Connect to the `shadow-cljs` nREPL:
    ```sh
    lein repl :connect localhost:8777
    ```
    The REPL prompt, `shadow.user=>`, indicates that is a Clojure REPL, not ClojureScript.

2. In the REPL, switch the session to this project's running build id, `:app`:
    ```clj
    (shadow.cljs.devtools.api/nrepl-select :app)
    ```
    The REPL prompt changes to `cljs.user=>`, indicating that this is now a ClojureScript REPL.
3. See [`user.cljs`](dev/cljs/user.cljs) for symbols that are immediately accessible in the REPL
without needing to `require`.

### Running Tests

Build the app with the `prod` profile, start a temporary local web server, launch headless
Chrome/Chromium, run tests, and stop the web server:

```sh
lein karma
```

Please be patient; it may take over 15 seconds to see any output, and over 25 seconds to complete.

### Running `shadow-cljs` Actions

See a list of [`shadow-cljs CLI`](https://shadow-cljs.github.io/docs/UsersGuide.html#_command_line)
actions:
```sh
lein run -m shadow.cljs.devtools.cli --help
```

Please be patient; it may take over 10 seconds to see any output. Also note that some actions shown
may not actually be supported, outputting "Unknown action." when run.

Run a shadow-cljs action on this project's build id (without the colon, just `app`):
```sh
lein run -m shadow.cljs.devtools.cli <action> app
```
### Debug Logging

The `debug?` variable in [`config.cljs`](src/cljs/todomvc/config.cljs) defaults to `true` in
[`dev`](#running-the-app) builds, and `false` in [`prod`](#production) builds.

Use `debug?` for logging or other tasks that should run only on `dev` builds:

```clj
(ns todomvc.example
  (:require [todomvc.config :as config])

(when config/debug?
  (println "This message will appear in the browser console only on dev builds."))
```

## Production

Build the app with the `prod` profile:

```sh
lein prod
```

Please be patient; it may take over 15 seconds to see any output, and over 30 seconds to complete.

The `resources/public/js/compiled` directory is created, containing the compiled `app.js` and
`manifest.edn` files.

The [`resources/public`](resources/public/) directory contains the complete, production web front
end of your app.

Always inspect the `resources/public/js/compiled` directory prior to deploying the app. Running any
`lein` alias in this project after `lein dev` will, at the very least, run `lein clean`, which
deletes this generated directory. Further, running `lein dev` will generate many, much larger
development versions of the files in this directory.
