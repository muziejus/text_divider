# text_divider

This is a Python implementation of David Hoover’s [Analyze Textual Divisions
Spreadsheet](https://wp.nyu.edu/exceltextanalysis/analyzetextualdivisions/).
The spreadsheet is a suite of macros that takes text that has been “lightly
marked-up” and creates a spreadsheet that categorizes each line in the text
file. This is a quick way to create divisions in a text (book, chapter,
section). More importantly, it allows for a light tagging scheme to define
dialogue in the text, giving a final file that lets you filter only one
character’s dialogue, for example.

## Markup syntax

Prof. Hoover’s markup is:

```
<1>    text division level 1
<2>    text division level 2
#<3>    text division level 3
#<4>    text division level 4
#[ ]       Letter writer
#{ }      Letter addressee
/          new speaker (character)
\          reporting clause (“speech marker”)
#>         copy without processing
#^          special character follows
```

Every tag that is commented out with a `#` is currently unimplemented. 

Dialogue, as noted above, is triggered by the `/`. All text between that and
the first `“` or `"` is understood to be the character’s name, and everything
after the quote marker is understood to be dialogue. It currently does not
support UK-style single-quoting, guillemets (`«»`), German-style low-9-quoting
(`„“`), or Russian/Joyce-style quotation dashes. It strikes me that converting
those on the fly to the pattern the system does understand can be done with a
vim macro (see below).

Dialogue ends with a blank line. Hence a line like:

```
“Hello,” said Alice, “And good-bye!” Then she walked away.
```

would be marked up as:

```
/Alice H.“Hello,”

\said Alice,

/Alice H.“And good-bye!”

Then she walked away.
```

With two handy vim macros (see below), breaking this up becomes rather easy. In
a speakers export, Alice H.’s dialogue would be concatenated into a file called
`aliceh.txt`.

Noting reporting clauses is done with a backslash, which lets them be separated
from the regular narrative, if you like.

The file
[`sample.txt`](https://github.com/muziejus/text_divider/blob/master/sample.txt)
is the file used for testing and also reveals how the markup can look in the
wild.

## Usage

For now, the usage is simply:

`python text_divider.py FILENAME [OUTPUT FILENAME]`

or you can install it with pip and then simply use:

`text_divider.py FILENAME [OUTPUT FILENAME]`

Of course,

`text_divider.py --help`

will include other tricks available from the command-line interface.

or you can get a bit fancier and make use of more methods using it as a module:

```
>>> import text_divider as td
>>> text = td.Text('sample.txt')
>>> speakers = text.all_speakers() # returns speakers sorted by lines of dialogue
>>> speakers
[(None, 18), ('Mr. Carraway', 3), ('Nick', 3), ('Daisy', 2), ('Tom', 2)]
>>> nick = text.speakers('Nick') # returns a string of all of the speaker’s dialogue
>>> nick
'The whole town is desolate. All the cars have the left rear wheel painted black as a mourning wreath, and there’s a persistent wail all night along the north shore.'
>>> text.export_top_speakers_to_txt(3, 'speakers_dir') # creates a “speakers_dir” and
# a separate text file for the top 3 speakers (including “none” for the narration)
# and collapses all the rest of the dialogue into a “minorspeakers.txt” file.
```

## Output

Using it with the command line gives a result similar to that of Prof. Hoover’s
original spreadsheet. It creates a *tab-delimited* `.csv` file where each line of text is
marked (or not) depending on what division it is in. For example, there might
be a “SPEAKER” column, and the value for the lines could be blank, “Mr. Carraway,” or
“Nick,” depending.

You can additionally pass the `--speakers-export` option with a path to a
directory, into which the program will place a separate `.txt` file for each
speaker. The same happens with `--sections-export`, but for sections.

Using it in the interpreter or within a program creates a list where each value
is a dictionary with keys as the column names. The current column names are:

```
text: the line of text
section: the names of the sections as a tree (for example, “Book I - Chapter 1”)
speaker: the speaker of the current line, if one is named, or other marking.
```

## Rationale

I couldn’t get Prof. Hoover’s spreadsheet to work on my computer, but I also
think that this tool makes it easy to feed customized text into
[`NLTK`](http://www.nltk.org). Specifically, I wanted a way to mark up a `.txt`
version of *[The Great Gastby](http://gutenberg.net.au/ebooks02/0200041.txt)*
so that I could programmatically get the dialogue on the fly. It strikes me
that the light markup of this program is more flexible and quicker to implement
than, say, creating a TEI version of the novel, etc., etc. 

## vim macros

Given a raw set of lines like:

```
"Whenever you feel like criticizing any one," he told me, "just remember that
all the people in this world haven't had the advantages that you've had."
```

I use two macros to move through this quickly. First, on the top line, I use:

```
^f"lli<Ret><Ret>\<Esc>{{ji/<Esc> 
```

This gets us to:

```
/"Whenever you feel like criticizing any one," 

\he told me, "just remember that
all the people in this world haven't had the advantages that you've had."
```

with the cursor right after the `/`, meaning that one can quickly type (or
paste) in the name of the speaker (`Mr. Carraway` in this case). Then you move
to the next line and fire the next macro:

```
J^f"i<Ret><Ret><Esc>o<Esc>ji/<Esc>
```

This gets us to:

```
/Mr. Carraway"Whenever you feel like criticizing any one," 

\he told me, 

/"just remember that all the people in this world haven't had the advantages that you've had."
```

With the cursor right by the second `/`, allowing you to type/paste in the name
again. The macros can be changed to use `“` or `”` instead of `"`. The script
recognizes both dialogue delimiters. For other quotation schemes, a quick
substitution would be necessary. In the case of guillemets, for example:

```
^:s/«/"/g<Ret>:s/»/"/g<Ret>f"lli<Ret><Ret>\<Esc>{{ji/<Esc> 
J^:s/«/"/g<Ret>:s/»/"/g<Ret>f"i<Ret><Ret>/<Esc>
```

## Copyright, etc.

This program is © 2016, Moacir P. de Sá Pereira. It is available under the GNU
GPL v. 3. See the `LICENSE` file for details.
