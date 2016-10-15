# text_divider

This is a Python implementation of David Hoover's [Analyze Textual Divisions
Spreadsheet](https://wp.nyu.edu/exceltextanalysis/analyzetextualdivisions/).
The spreadsheet is a suite of macros that takes text that has been “lightly
marked-up” and creates a spreadsheet that categorizes each line in the text
file. This is a quick way to create divisions in a text (book, chapter,
section). More importantly, it allows for a light tagging scheme to define
dialogue in the text, giving a final file that lets you filter only one
character’s dialogue, for example.

## Markup syntax

Hoover’s markup is:

```
<1>    text division level 1
#<2>    text division level 2
#<3>    text division level 3
#<4>    text division level 4
#[ ]       Letter writer
#{ }      Letter addressee
/          new speaker (character)
#\          speech marker
#>         copy without processing
#^          special character follows
```

Every tag that is “commented out” with a `#` is currently unimplemented. 

The file `sample.txt` is the file used for testing and also reveals how the
markup can look in the wild.

## Usage

For now, the usage is simply:

`python text_divider.py FILENAME [OUTPUT FILENAME]`

or you can install it with pip and then simply use:

`text_divider.py FILENAME [OUTPUT FILENAME]`

or

```
>>> import text_divider as td
>>> divided_text = td.parse('FILENAME')
>>> speakers = divided_text.all_speakers() # returns speakers and the lines of dialogue
>>> print(speakers)
[('Mr. Carraway', 3), ('Tom', 2), ('Daisy', 2), (None, 18), ('Nick', 3)]
>>> nick = divided_text.speakers('Nick') # returns a string of all of the speaker’s dialogue
'The whole town is desolate. All the cars have the left rear wheel painted black as a mourning wreath, and there’s a persistent wail all night along the north shore.'
```

## Output

Using it with the command line gives a result similar to that of the original
spreadsheet. It creates a `.csv` file where each line of text is marked (or
not) depending on what division it is in. For example, there might be a
“Dialogue” column, and the value for the lines could be blank, “Alice,” or
“Bob,” depending.

Using it in the interpreter or within a program creates a list where each value
is a dictionary with keys as the column names. The current column names are:

```
text: the line of text
```

## Rationale

I couldn’t get Prof. Hoover’s spreadsheet to work on my computer, but I also
think that this tool makes it easy to feed customized text into
`[NLTK](http://www.nltk.org)`. Specifically, I wanted a way to mark up a `.txt`
version of *[The Great Gastby](http://gutenberg.net.au/ebooks02/0200041.txt)*
so that I could programmatically get the dialogue on the fly. It strikes me
that the light markup of this program is more flexible and quicker to implement
than, say, creating a TEI version of the novel, etc., etc. 

## Copyright, etc.

This program is © 2016, Moacir P. de Sá Pereira. It is available under the GNU
GPL v. 3. See the `LICENSE` file for details.
