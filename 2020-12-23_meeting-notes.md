## Sandbox

- A separate repository for:
	-  Experimenting with ideas 
	- Fleshing out proposals
	- Comparing alternative implementations
	- etc.
- Incomplete/rejected features should be preserved for posterity
- Should this be a separate repo or is it better to use feature branches for this?


## Directory layout (reprise)

One possibility:
```
caesar
├── ci/...
├── cxx
│   ├── cmake/...
│   ├── docs/...
│   ├── src/...
│   ├── test/...
│   └── CMakeLists.txt
├── python
│   ├── caesar/...
│   ├── docs/...
│   ├── ext/...
│   ├── test/...
│   └── setup.py
├── LICENSE
├── README.md
└── VERSION
```


## Banned words

Don't use these in class names, filenames, etc:
- tools
- util
- misc
- {...}manager
- {...}class
- do


## Default git branch

"main"


## Documentation

- Components
	- Installation guide
	- Getting started
	- Tutorials / code snippets
	- API Reference
	- Contributing guide
- C++
	- Doxygen
	- Doxygen + Breathe + Sphinx
	- Don't have auto-generated documentation from inline comments
- Python
	- Sphinx
- reStructuredText vs Markdown
	- Markdown seems to be simpler; ReST seems to be more extensible and feature-rich
	- Doxygen optionally supports Markdown; Sphinx uses ReST
- GitHub Pages vs ReadTheDocs
	- GH Pages more flexible


## Default symbol visibility

- Setting default symbol visibility to hidden:
	- may reduce binary size
	- prevents users from relying on symbols which aren't part of the API
	- requires you to add an attribute (`CAESAR_EXPORT`) to each symbol that you want to export
- This is the default in MSVC (Clang & GCC require specifying `-fvisibility=hidden`)
- Experiment with this later on and revisit


## Output parameters

- Return values should be preferred over output parameters
- If output parameters are used, they should be the first argument(s) that the function takes, and ~~should be passed by pointer (never by non-const reference)~~ (evaluate this on a case-by-case basis and revisit this judgement based on experience)
