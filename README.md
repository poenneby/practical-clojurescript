# Practical Clo*j*ureScript

## Intro

This is a guided hands-on which let you experience building a simple front end to the API built in [practical clojure](https://github.com/poenneby/practical-clojure).

## Setup your development environment

- You need Java 8 or above installed and `JAVA_HOME` set.

- Install Leiningen: https://leiningen.org/#install or `brew install leiningen`

### Editor/IDE
- For Visual Studio Code users I recommend installing the [Calva plugin](https://marketplace.visualstudio.com/items?itemName=cospaia.clojure4vscode) - setup instructions [here](https://github.com/BetterThanTomorrow/calva/wiki/Getting-Started#dependencies)

- If you use IntelliJ I recommend installing the [Cursive plugin](https://plugins.jetbrains.com/plugin/8090-cursive) - and there is a free [non-commercial licence](https://cursive-ide.com/buy.html) available for hacking and open source work.

- Vim users have a number of useful plugins
  - [kien/rainbow_parentheses.vim](https://github.com/kien/rainbow_parentheses.vim)
  - [guns/vim-clojure-static](https://github.com/guns/vim-clojure-static)
  - [tpope/vim-fireplace](https://github.com/tpope/vim-fireplace)
  - [vim-scripts/paredit.vim](https://github.com/vim-scripts/paredit.vim)

- Emacs users can have a look at this
  - http://clojure-doc.org/articles/tutorials/emacs.html

## The application

Let's imagine we've been asked to build a website that expose information related to French monuments.

Since we have already built the [backend](https://github.com/poenneby/practical-clojure/tree/solution), "all" that is left for us to do is building the front end and for that we will use ClojureScript.

After a bit of discussion with our PO we agree on starting out simple with
an input field for searching, and that we can show more information about each monument found.

*In the [practical clojure](https://github.com/poenneby/practical-clojure/tree/solution) tutorial we built the REST API that allow searching for monuments by region.*

## Start the backend API

Go ahead and clone [practical clojure solution](https://github.com/poenneby/practical-clojure/tree/solution) and run it

```
git clone https://github.com/poenneby/practical-clojure.git
cd practical-clojure
git checkout solution
cd monumental
lein ring server
```

## Generate a project

The Clojure community has been quick to see the potential in React and there exist a few different wrappers libraries.
By far the most popular is [Reagent](http://reagent-project.github.io/) which allows you to declare your components using [Hiccup](http://weavejester.github.io/hiccup/)

The Reagent project conveniently provide a [Leiningen template](https://github.com/reagent-project/reagent-template) and we can generate a new project like this.

`lein new reagent monumental-front`

The project provides all the basic needs of a front end project such as routing and live reloading of JS and CSS.

```
> cd monumental-front
> tree
.
├── LICENSE
├── Procfile
├── README.md
├── env
│   ├── dev
│   │   ├── clj
│   │   │   ├── monumental_front
│   │   │   │   ├── middleware.clj
│   │   │   │   └── repl.clj
│   │   │   └── user.clj
│   │   └── cljs
│   │       └── monumental_front
│   │           └── dev.cljs
│   └── prod
│       ├── clj
│       │   └── monumental_front
│       │       └── middleware.clj
│       └── cljs
│           └── monumental_front
│               └── prod.cljs
├── project.clj
├── resources
│   └── public
│       └── css
│           └── site.css
├── src
│   ├── clj
│   │   └── monumental_front
│   │       ├── handler.clj
│   │       └── server.clj
│   ├── cljc
│   │   └── monumental_front
│   │       └── util.cljc
│   └── cljs
│       └── monumental_front
│           └── core.cljs
└── system.properties

21 directories, 16 files
```

Yes there are a lot files there ... but you can ignore most of them for now.

You'll notice that Clojure source files are divided into "server" `src/clj`, "common" `src/cljc` and "front" `src/cljs`. Fullstack Clojure projects can
thus conveniently share code between the server and the frontend.

The files that interest us for this tutorial reside in `resources/public/css` and `src/cljs/monumental_front`

Open up the project in your favourite editor and start [Figwheel](https://github.com/bhauman/lein-figwheel)

```
> lein figwheel
Figwheel: Cutting some fruit, just a sec ...
```

This starts up a server listening on port 3449 and a REPL watching your files for changes.

Open the page in a navigator at `http://localhost:3449` and have a look around.

## A code overview

Open up `src/cljs/monumental_front/core.cljs`

Starting from the bottom of the file we see the `init!` function that setup the basic routing and page navigation before mounting the root component in `mount-root`.

For **React** developers the code below should be pretty familiar:

```clojure
(defn mount-root []
  (reagent/render [current-page] (.getElementById js/document "app")))
```

`current-page` is a component and it gets rendered in the element identified by `app`.

The `current-page` function calculates the current page from the path via Reagent's session abstraction.

The rest is template code using **Hiccup** to generate HTML.

Working our way up in the file we find a function `page-for` translating routes to functions (page components)

The var quote form `#'home-page` is short for `(var home-page)` which expands into the fully qualified function name `monumental-front.core/home-page`

Further up we have some pages declared as functions using **Hiccup**. All the declared pages return a function and this is optional.
You could just return a vector directly and this would translate to the `render` function of a React class.
However if you want to do stuff like setting up initial state or fetching data prior to rendering the component you do that in the top level function.

Finally at the top we have the declaration of routes. The Reagent template is using [reitit](https://metosin.github.io/reitit/) which claims to be a very [fast](https://metosin.github.io/reitit/performance.html) routing library for Clojure(Script).

An added advantage is that routes can be shared between server and client.

## Clean up

Now we move on to building our front end - let's start by removing some unnecessary code.

We start from the bottom again working our way up. The `current-page` function contain a navigation header and a footer that we don't need.

Remove these and you will end up with a function looking like this:

```clojure
(defn current-page []
  (fn []
    (let [page (:current-page (session/get :route))]
      [:div
       [page]])))
```

We remove the "about" page: the reference in `page-for`, the `about-page` page component and the route declarations at the top.

(Remember to keep an eye on the rendered page to make sure you didn't remove too much code causing the page to no longer compile!)

In the `home-page` page component we remove the "borken link", the link to "items" and we change the title to "Monumental".

```clojure
(defn home-page []
  (fn []
    [:span.main
     [:h1 "Monumental"]]))
```

You can also remove the "item-page" and "items-page" along with their references in the `router` declaration.

```
(def router
  (reitit/router
   [["/" :index]]))
```

## Introducing state

Right at the top we see the import of `atom` - Atoms are used to [manage state](http://reagent-project.github.io/docs/master/ManagingState.html) in Reagent applications.

```clojure
(ns monumental-front.core
    (:require [reagent.core :as reagent :refer [atom]]
              [reagent.session :as session]
              [reitit.frontend :as reitit]
              [clerk.core :as clerk]
              [accountant.core :as accountant]))
```

Declare an atom containing a map with `monuments` and the current `search`
```clojure
(defonce state (atom {:monuments []
                  :search ""}))
```

`monuments` is an empty vector for now but this is where we'll put the data retrieved from the web service later on.


## The search component

We need a search component to search for monuments so let's add it to the home page component:

```clojure
(defn home-page []
  (fn []
    [:span.main
     [:h1 "Monumental"]
     [<search>]]))
```

**Figwheel** now give us a compile warning in the browser indicating that the variable is not declared. Let's fix that.

```clojure
(defn <search> []
  [:div [:input {:placeholder "Search by region ..."}]])
```

But in order for it to do something we bind the onchange event to fetch monuments.

```clojure
(defn <search> []
  [:div [:input
         {:placeholder "Search by region ..."
          :on-change #(fetch-monuments (-> % .-target .-value))}]])
```

The `#(...)` is short hand for an anonymous function (`(fn [e] e)`) and the `%` represents the parameter - a javascript `event` -
and we access the value of the event via the javascript interoperability. The thread-first macro `->` makes accessing the target value more readable.
Compare the above with the equivalent expression without using the thread-first macro:
```clojure
(.value (.-target (%)))
```

Now we get a compiler error as the `fetch-monuments` function is not yet defined.

```clojure
(defn fetch-monuments [region]
  (swap! state assoc :search region))
```

We associate the current search to the `:search` keyword in the state atom. Finally we associate the search state to the inputs' value:

```clojure
(defn <search> []
  [:div [:input
         {:placeholder "Search by region ..."
          :value (:search @state)
          :on-change #(fetch-monuments (-> % .-target .-value))}]])
```

We can make that input a little prettier by adding some styling in `resources/public/site.css`

```css
input {
  width: 200px;
  padding: 10px;
  font-size: 18px;
}
```

## Fetching monuments

Now that we have a way to input search terms we can focus on fetching the monuments.

For that we need to add a dependency to `project.clj`

Locate the `:dependencies` key and add the following dependency at the end of the vector:
`[cljs-ajax "0.8.0"]`

When modifying the project configuration we need to restart the REPL so quit with `Ctrl+D` and start it again with `lein figwheel`

`cljs-ajax` is a simple Ajax client for ClojureScript and Clojure and it exposes different functions representing HTTP methods such as `GET` and `POST`.

Let's try it out while in the REPL.

First import the module and the `GET` function:

```clojure
app:cljs.user=> (require '[ajax.core :refer [GET]])
```

Then we can use the `GET` function:

```clojure
app:cljs.user=> (GET "http://localhost:3000/api/search" {:params {:region "P"}
           #_=>                                          :handler #(.log js/console (str %))})
```

We call the search endpoint with a query parameter `region` and the handler function logs to the console.

Open up your navigator's devtools and you should see the results of the `GET` in the console.

Furthermore we will hint that the response type is json and that we want `keywords` instead of the JSON string keys in the data -
this is a common data format to work with in Clojure.

```clojure
app:cljs.user=> (GET "http://localhost:3000/api/search" {:params {:region "P"}
           #_=>                                          :response-format :json
           #_=>                                          :keywords? true
           #_=>                                          :handler #(.log js/console (str %))})
```

Verify in the navigator console and you should see an output similar to below:

```edn
[{:AFFE "", :STAT "Propriété d'une personne privée", :REF "PA00109152" ...]
```

Since this code is now working we'll put it in the `fetch-monuments` function, but instead of logging the response we put it in the state atom.

```clojure
;; Add the dependency in the require vector at the top of the file
[ajax.core :refer [GET]]


(defn fetch-monuments [region]
  (swap! state assoc :search region)
  (GET "http://localhost:3000/api/search" {:params {:region region}
                                           :response-format :json
                                           :keywords? true
                                           :handler #(swap! state assoc :monuments %)}))
```

## Listing the monuments

With the monuments fetched we can focus on listing them.

We define 2 new components

```clojure
(defn <monument-line> [monument]
  (let [current-name (:TICO monument)
        monument-ref (:REF monument)
        region (:REG monument)]
    [:li {:key monument-ref} current-name ", " region " "
     [:a {:href (path-for :monument {:monument-id monument-ref})} "more..."]]))

(defn <monuments> [] [:div.monuments
                      [:ul
                       (for [monument (:monuments @state)] (<monument-line> monument))]])
```

The `<monuments>` component iterate over the `monuments` in the state atom using "for comprehension" - For each `monument` we render a `<monument-line>`.

The `<monument-line>` component first extract variables from the `monument` and then generate a list item with a link to more info.

We then reference the `<monuments>` component from the home page component

```clojure
(defn home-page []
  (fn []
    [:span.main
     [:h1 "Monumental"]
     [<search>]
     [<monuments>]]))
```

You will notice that the links aren't working. This is because we haven't declared the `monument` routes. Let's fix that:

```clojure
(def router
  (reitit/router
   [["/" :index]
    ["/monuments/:monument-id" :monument]]))

...

(defn page-for [route]
  (case route
    :index #'home-page
    :monument #'monument-page
    ))
```

And now on to the `monument-page`:

```clojure
...

(defn monument-matching-ref [monuments ref]
  (filter (fn [monument] (= ref (:REF monument))) monuments))

...

(defn <monument> [monument]
  (let [current-name (:TICO monument)
        details (:PPRO monument)]
    [:div
     [:h1 current-name]
     [:div details]
     [:p [:a {:href (path-for :index)} "Back to the list of monuments"]]
     ]))

...

(defn monument-page []
  (fn []
    (let [routing-data (session/get :route)
          monument-id (get-in routing-data [:route-params :monument-id])
          monument (first (monument-matching-ref (:monuments @state) monument-id))
          ]
      [:span.main
        [<monument> monument]])))

```

In `monument-page` we get the id of the monument from the url. This id is then used to filter out the `first` monument matching the reference.

The `<monument>` component extract variables from the monument and render the information.

To make things a little more interesting we can display a photo of the monument.

```clojure

;; Add the "lower-case" function dependency in the require vector at the top of the file
[clojure.string :refer [lower-case]]


(defn <monument> [monument]
  (let [current-name (:TICO monument)
        details (:PPRO monument)
        department (:DPT monument)
        reference (:REF monument)]
    [:div
     [:h1 current-name]
     [:div details]
     [:div [:img {:src (str "https://monumentum.fr/photo/" department "/"  (lower-case reference) ".jpg")}]]
     [:p [:a {:href (path-for :index)} "Back to the list of monuments"]]
     ]))

```

And here's some CSS to make the image fit better

`resources/public/css/site.css`
```css
img {
  height: 60%;
  width: 60%;
}
```
## Distributing our application

We are ready to ship the code and we do so by generating an **uberjar**:

`> lein do clean, uberjar`

The uberjar contains all the application's dependencies and you can run it like this:

`java -jar <path to generated jar>`

Ex. `> port=9000 java -jar target/monumental-front.jar`

## Conclusion

Congratulations, you've built a web application for looking up French monuments with ClojureScript and Reagent!

We are of course only scratching the surface of what is possible using Clojure(Script) but I hope you have
now got an idea of what you can achieve with relatively little code. And that you will want to continue to learn this programming language.
