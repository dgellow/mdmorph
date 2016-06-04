# mdmorph

A `Renderer` protocol and some functions so export a [markdown2clj](https://github.com/nilenso/markdown2clj) AST in various format.

[](dependency)
```clojure
[dgellow/mdmorph "1.0.0"] ;; latest release
```
[](/dependency)

```clj
(require '[markdown2clj.core :as md]))
(require '[dgellow.mdmorph.core :as morph]))

(def document
  (->> "documentation.md"
       clojure.java.io/resource
       slurp
       md/parse
       :document))

# Default render functions
(morph/to-html document)
(morph/to-ansi document)

# Render every segments of markdown2clj's AST using a Renderer
(->> document
  (map (partial morph/render-segment (morph/HtmlRenderer.)))
  (clojure.string/join "\n"))
```

## The `Renderer` protocol

```clj
(defprotocol Renderer
  (heading [_ segment])
  (paragraph [_ segment])
  (fenced-code-block [_ segment])
  (text [_ segment])
  (link [_ segment])
  (soft-line-break [_ segment])
  (code [_ segment])
  (bold [_ segment])
  (italic [_ segment]))
```

Example of implementation with the `HtmlRenderer`:

```clj
(defrecord HtmlRenderer []
  Renderer
  (heading [renderer [{level :level} & more]]
    (->> more
       (map (partial render-segment renderer))
       clojure.string/join
       ((fn [x] (format "<h%s>%s</h%s>" level x level)))))
  (paragraph [renderer segment]
    (->> segment
       (map (partial render-segment renderer))
       clojure.string/join
       (format "<p>%s</p>")))
  (fenced-code-block [_ segment]
    (->> (some :text segment)
       ((fn [x] (format "<pre>%s</pre>" x)))))
  (text [_ text] text)
  (link [_ [{title :title} {dest :destination} {text :text}]]
    (str "<a"
         (format " href=\"%s\"" dest)
         (when title (format " title=\"%s\"" title))
         ">" text "</a>"))
  (soft-line-break [_ _] "\n")
  (code [_ [{text :text}]]
    (format "<code class=\"inline\">%s</code>" text))
  (bold [_ [{text :text}]]
    (format "<strong>%s</strong>" text))
  (italic [_ [{text :text}]]
    (format "<em>%s</em>" text)))
```

You can call the renderer by using `morph/render-segment` or you can implement your own.
