# Unknown

Building great documentation for (especially open source) projects or
writing blog posts on development topics brings you to the limits of
Markdown. Unknown adds custom filters to documents and code
examples withing Markdown to:

* Display input and output of some operation
* Add additional information (like a file name)
* Integrates pre processors

It is already heavily used to generate front-end style guides with the
[LivingStyleGuide Gem](http://github.com/hagenburger/livingstyleguide#readme)
and was originally created within this project.

The syntax is inspired by CSS, Sass/SCSS, different code documentation tools,
and Cucumber options. It should be attractive to front-end developers from
both—the JSON/SCSS/HTML/JavaScript and the Ruby/YAML/Sass/Haml/Coffee-Script
world.


## Parsing

Parse and execute filters on documents.

``` ruby
document = Unknown.parse(string)
document.render
```

Returs a string (mostly HTML) of the filtered document.


## Standard Filters

### Setting Options

All three examples have the same result:

```
@set source-type: haml
@set haml/attr-wrapper: "
```

```
@set yaml
  source-type: haml
  haml:
    attr-wrapper: "
```

```
@set json {
  "source-type": "haml",
  "haml": { "attr-wrapper": "\"" }
}
```


### Defining Data

```
@data name: Homer
@data city: Springfield
```

```
@data yaml
  name: Homer
  city: Springfield
```

```
@data json {
  "name": "Homer",
  "city": "Springfield"
}
```


### Importing Content

```
<h1>Hello World</h1>
```

Same result as:

```
@content hello-world.html
```

(If _hello-world.html_ contains `<h1>Hello World</h1>`)

* If the suffix differs from `.html`, a pre processor will be called if
  defined.
* The pre processor can access [the data](#defining-data) if defined.


## Adding Filters

``` ruby
Unknown.add_filter :replace do |arguments, data|
  content do |content|
    content.gsub(Regexp.new(arguments), data)
  end
end
```

The `@replace` filter becomse available:

```
@replace /world/: moon
Hello world!
```

Which results in:

```
Hello moon!
```


## Adding Data Types

``` ruby
Unknown.add_data_type :pipe_separated_content do |data|
  data.split('|')
end
```

This will allow use the new data type:

```
@data pipe-separated-content {
  Homer | Marge | Lisa | Maggie | Bart
}
```

Most template engines expect hashes and won’t work with arrays;
so this example should be wrapped:

``` ruby
Unknown.add_data_type :pipe_separated_content do |data|
  {
    simpsons: data.split('|')
  }
end
```


## Adding Blocks

By default only one output block is definden (`content`).
You can create addional blocks (e.g. for source code, information, …)
by defining them:

``` ruby
Unknown.add_block :source do |block|
  block.attributes['data-type'] = @options[:source_type]

  source do |content|
    syntax_highlight(content, @options[:source_type])
  end
end
```

This will automatically create the block. You don’t have to add anything
to your document.


## Adding Pre Processors

A pre processor will convert the source into the output type (usually
HTML). The original source is still accessible and can be outputted into
another block (e.g. for syntax-highlighted source code).

``` ruby
Unknown.add_pre_processor :haml do
  @source_type = :haml

  pre_processor do |source|
    Haml::Engine.new(content, options[:haml]).render.strip
  end
end
```

* Creates a pre processor
* Parses *.haml when [imported](#importing-content)
* Creates a filter @haml:
  Unknown.add_filter :haml do |arguments, data|
    @options[:source_type] = :haml
    data_format = arguments.to_sym
    @options[:haml] = parse_data(data, data_format)
  end
* Inits @options[:haml] = {}

So you can use either:

```
@haml
%h1
  Hello World!
```

Or:

```
@import hello-world.haml
```

