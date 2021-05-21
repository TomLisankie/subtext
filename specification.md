# Subtext

## Speculative specification

**Version**: 2021.05.17.dev  
**Status**: Draft  
**Authors**: Gordon Brander  

This is a rough sketch of an actual spec for Subtext. It is becoming increasingly less rough over multiple passes. This doc is being shared in the spirit of working with the garage door open. Feedback is welcome!

**Warning to implementors**: Subtext is a hypothesis, an experiment, NOT a finished language proposal. We are actively putting it through its paces using practical prototypes. It may change in radical and breaking ways between now and a first stable release. 

## Overview

Subtext is a text-based, line-oriented hypertext format. It is designed with note-taking in mind. It has a simple, people-friendly syntax that is easy to read, and difficult to mess up.

Subtext markup is made up of a sequence of lines. Lines that are prefixed with "magic" characters are treated as specially tagged blocks. Lines without leading special characters are tagged as text blocks. Empty lines are ignored. Here's a quick sample:

```
# Heading

Plain text

- List item
- List item

> Quoted text

& example.csv
& https://example.com
```

Link blocks (lines starting with `&`) allow you to link to other files within the flow of a Subtext document. Any kind of file can be linked, including other Subtext documents! This makes Subtext composable, and gives it hypertext properties similar to Ted Nelson's ELF format (Nelson, 1965).

Subtext is designed to be used in a wide range of settings, from simple command-line utilities, to advanced clients. It is extremely easy to parse, and can be rendered in a single pass, or even streamed.

## Markup for note-taking

Subtext is designed with hypertext note-taking in mind. It represents _block-oriented documents_ as _line-oriented markup_.

A typical block-oriented document is made up of a list of blocks of different types (or occasionally, a tree of blocks). Each block type may be displayed differently. For example, a quote block may render as quote-formatted text, while an image block may render an image in-place.

Some of the earliest hypertext proposals were block-oriented, including Ted Nelson's ELF (Nelson, 1965). Block-oriented documents have also independently evolved within many contemporary tools-for-thought, including [Notion](https://www.notion.so/), [Roam](https://roamresearch.com/), and [Ward Cunningham's Federated Wiki](http://fed.wiki.org/view/federated-wiki).

Block-oriented documents allow for a rich mix of text, images, and other formatted content, while remaining easy to understand. Block-oriented documents are also composable (and decomposable). You can break them apart into component blocks, filter down to blocks of a particular type, merge documents, pluck out blocks, link to specific blocks, etc. Working with documents programmatically is easy because the document structure is linear. Try  meaningfully merging two HTML files... tag soup!

Subtext takes the magic of _block-oriented documents_ and represents it as _line-oriented markup_, with each line describing a block. Plain text is simple, human-readable, and most importantly, it will never go away. You can work with plain text using a wide range of tools, from GUI editors, to grep. Subtext takes universal plain text, and layers a bit of structure on top, in a way that still make sense to read as plain text.

I'm developing Subtext as part of a tool-for-thought, called [Subconscious](https://subconscious.substack.com/). I expect Subtext could be broadly useful for other note apps, wikis, [Zettelkasten](https://en.wikipedia.org/wiki/Zettelkasten), or plain-text note-taking. Subtext is useful anywhere a bit of structure might help you (and your computer) think more meaningfully with text.


## Design goals

### People goals

- **People-friendly**. We've settled on a syntax that is as close to plain text as we could manage. It has a passing resemblance to Markdown.
- **Hand-writeable**. We want to avoid imposing too much syntax on the user who is hand-editing Subtext. It should be possible to write ordinary lines of plain text, prose, and get a working document out of it.
- **Simple and mistake-proof**. At this time, it is not possible to write invalid Subtext documents by hand. There are no brackets to mis-type or closing tags to omit. We would like to retain this property as much as possible, even if Subtext's features grow in the future.

### Computer goals

- **Easy to implement**. Clients can implement features that make sense for their use-cases, and fall back to plain text for unsupported features.
- **Composable and decomposable into component blocks**. You can compose documents from many small pieces, including from other Subtext documents.
- **Simple to meaningfully manipulate with software**. With a linear data model, the range of meaningful document structures is narrowed. this means you can make complex, yet meaningful programmatic decisions about how to manipulate it without having to understand as much about the structure and context of the specific document.

Using Subtext, you should be able to programmatically do things like:

- Excerpt documents by showing only the first text block
- Append this document to that one
- Select all quotes from a collection of documents
- Autogenerate a table of contents from all heading blocks in a document
- Select all links, and generate a graph for a collection of documents
- Find all backlinks and append them to the document as links

### Non-goals

Subtext is not a layout, presentation, or word processing format. The analogy to draw here is index cards, not books or manuscripts. Subtext deliberately avoids the kind of complex presentation features offered by publishing formats like HTML, PDF, and LaTex. It has no opinions about fonts, colors, sizes. It does not deal in scripting behavior.

We picture Subtext being used for note-taking systems, like as [Zettelkasten](https://en.wikipedia.org/wiki/Zettelkasten). A publishing workflow might look like copy-pasting Subtext into a publishing tool to create an artifact, or perhaps exporting Subtext into another format, like HTML, and then editing it into shape.

This puts Subtext in a different category than complex markup languages like HTML5. It is closer in spirit to something like RSS or OPML. Subtext brings a minimal amount of semantic structure to linear documents. It is up to clients to decide how that structure is used and displayed.

## Sample document

```
# Overview

Evolution is a behavior that emerges in any system with:

- Mutation
- Heredity
- Selection

Evolutionary systems often generate unexpected solutions. Nature selects for good enough.

> There is no such thing as advantageous in a general sense. There is only advantageous for the circumstances you’re living in. (Olivia Judson, Santa Fe Institute)

Evolving systems exist in punctuated equilibrium.

& punctuated-equilibrium.st

# Questions

- What systems (beside biology) exhibit evolutionary behavior? Remember, evolution happens in any system with mutation, heredity, selection.
- What happens to an evolutionary system when you remove mutation? Heredity? Selection?
- Do you see a system with one of these properties? How can you introduce the other two?

# See also

& https://en.wikipedia.org/wiki/Evolutionary_systems
```

## Line-orientation

Subtext is line-oriented. Lines can be parsed to blocks by checking if the leading characters of the line match one of a set of special character sequences. These special character sequences are called "sigils", and determine the block type of the line.

All sigils currently happen to be two characters: one character plus a space. In future, we may introduce multi-character sigils, so implementors MUST NOT determine block type by sampling only first two characters. However, we will try to avoid multi-character sigils for reasons of aesthetics.

**TODO**: fix an upper limit on sigil length?

Note that the space after sigil characters is part of the sigil and is not optional. If the sigil character does not end in a space, it is not a sigil character.

```
# This is a heading
#This is just text with a weird character in front.
```

## Core block types

### Blank lines

Blank lines MUST be ignored, i.e. they MUST NOT be represented in parsed Subtext output. They MAY be added to the source code of documents for the purpose of readability, but are optional.

### Text blocks

Text blocks are the fundamental line type. Any line which does not match the definition of another line type defaults to being a text line.

Text lines SHOULD be presented to the user in a manner suitable for general reading. This SHOULD include wrapping text, where appropriate. Text lines MAY be formatted for reading at the client's discretion. For example, clients MAY choose to display text with variable fonts, leading and font-size appropriate to the context. 

### Link blocks

Link blocks start with `& `. They reference other files within the flow of a Subtext document. Any kind of file can be linked, including other Subtext documents.

The client MUST render some kind of user-interactable link to the document. The client MAY choose what form this link takes.

For file types it understands, the client MAY render all or part of the linked document in-place (e.g. translude). For example:

- A linked image file (`.png`, `.jpg`, `.gif`, etc) MAY be rendered in-place by the client and sized to fit its context.
- A linked video file MAY be rendered in-place, together with playback controls.
- A linked `.csv` file MAY be rendered in-place as a table.
- A linked Subtext file MAY be transcluded (linked while rendered in-place), rendered in-place in full, or excerpted and rendered in-place, depending on the use-case.

Links are the most important feature in Subtext. By allowing you to reference other documents, you can compose hypertext documents from many smaller documents.

Rather than extending the syntax of Subtext to include features like tables, videos, or deeply nested lists, our sense is that a hypertext format allows these special types to be represented in their native file containers. Clients that understand these other file types MAY embed them, or even allow you to edit them in-place. This keeps Subtext simple, and allows data sources like `.csv`, or `.png` to be OPTIONALLY embedded in-place, while remaining valid file types that can be opened, edited, and used in best-of-breed applications.

**For future exploration**: Multiple sources. We could support providing multiple link sources through a space-separated list of URIs. A client could choose whichever one is best. This could allow for the coexistence of `http://`, and `ipfs://` links, for example.

**For future exploration**: Citations via DOI links. Citations could be embedded as DOIs. The recommended format for DOIs since 2011 is an active link (for example `http://doi.org/10.3352/jeehp.2013.10.3`). Adding DOI links could allow clients to automatically compile citations from a Subtext document.

**For future exploration**: Search links. "Any sufficiently advanced search is indistinguishable from a hyperlink". Rather than linking to file paths or URLs, links could include a search string, allowing a client to choose the best match from an index. This "search link" approach is inspired by [Notational Velocity's ](https://notational.net/) treatment of `[[WikiLinks]]` as search strings.

### Heading blocks

Heading blocks start with `# `.

Heading blocks SHOULD be presented in a manner denotes that they are hierarchically "above", and "label" blocks below. This MAY   mean typical typographic heading treatment in visual clients, such as increasing the font size as compared to text blocks, or displaying the text in bold. In non-visual clients, such as screen readers, this MAY mean announcing the block using a different voice style.

Clients MAY also create navigational affordances for headings, such as deriving a Table of Contents from heading blocks, or creating jump points in the scroll bar.

Subtext currently supports only one level of heading. This is a deliberate design choice, since a deep heading hierarchy is probably a sign your note needs to be refactored or unbundled into multiple notes. To construct deeply nested documents, you can link to other notes using a link block, instead. Supporting clients MAY read in these nested documents, embed them in-place, and adjust heading sizes as necessary, to denote heading level. Solving hierarchy through links encourages a hypertext method of writing, and allows pieces of a document to be used in more than one place.

### List blocks

List blocks start with `- `. List blocks SHOULD be presented in a manner that denotes they are lists. Visual clients MAY render list blocks with a bullet preceding. They MAY also choose to visually group contiguous list blocks together, for example, by removing the margins between them.

Subtext currently supports only one level of list. Our sense at this time is that the benefits of deep lists do not outweigh the costs of complicating the document format by introducing hierarchy. Clients MAY consider transcluding links to formats like `.yaml`, `.json`, or `.opml` to display deep lists.

### Quote blocks

Quote blocks start with `> `. Quote blocks SHOULD be presented in a manner that denotes they are quoted text. Visual clients MAY render quote blocks by indenting them, or by rendering them with a line to the left, as seen in many email clients. Non-visual clients, such as screen readers MAY read quote blocks in a different voice style.

## Sigils reserved for future use

The following sigils are reserved for possible future use:

- `$ `
- `@ `
- `! `
- `% `
- `~ `
- `| `
- `: `
- `* `
- `+ `
- `= `
- `\ `
- `λ `
- `\s\s` (two or more contiguous literal space characters leading a line)
- `---`

Note, again, that the sigil includes the space character.

## Mime Type and extension

The preferred file extension for Subtext is `.st`.

The Mime Type for Subtext is `text/subtext`. As a subtype of the top-level media type "text", "text/subtext" inherits the "charset" parameter defined in [RFC 2046](https://tools.ietf.org/html/rfc2046). However, the default value of "charset" is "UTF-8" for Subtext "text" content.

Subtext is valid plain text and plain text is, in most cases, valid Subtext. In most cases it should be practically possible to read `.txt` files as Subtext, if a client wishes to.

## Appendix 1: Future explorations

### Key-value blocks

We could explore expanding Subtext to support markup for [key-value pairs](https://en.wikipedia.org/wiki/Attribute%E2%80%93value_pair).

```
Q: What is Subtext?
A: Subtext is a markup language for note-taking.
```

A key-value block is any alphanumeric string followed by a `:`. The alphanumeric string before the `:` becomes the sigil type for the block.

Sigil, described as a regular expression:

```
^[a-zA-Z0-9_]+:\s
```

Key-value pairs are a fundamental primitive with a wide range of potential use-cases for tooling. Like any other type of block, key-value blocks could be gathered by key into lists, concatenated, or collected using a first- or last-key-wins to get simple key/value data.

- You could execute queries such as: “list all questions (`Q:` blocks) in my notes”.
- You could transform a collection of notes into a sparse table by treating each note as a row, and treating keys as columns. Denser tabular data can be had by filtering notes to only include those with a particular set of keys, and then concatenating, JSON-encoding, or dropping duplicate keys. Tada! CSV.
- You could include headmatter in the body of a note. This can make it easier to integrate notes with static site generators, or other tools.

**Open question**: what are the implications for parsing? It would require us to run a search on across a string for an unbounded number of characters, until we encounter a space character, before defining the block as a text block. That means this search must happen to every block before it can be found to be a text block. Is this a problem in practice? Are there ways we could simplify this algorithmically?

### Hashtags

We could explore expanding Subtext to include support for hashtags in text blocks:

```
There’s no such thing as advantageous in a general sense. It’s advantageous in the circumstances you’re living in. #evolution #ecology
```

Hashtags could be collected into a "tags" field of text blocks, and perhaps stripped from the text as well.

**OTOH**: Tags are just a backlinks to pages that don't exist. To use an OODA lens, tagging is O without ODA. Collecting into tagged buckets is not sense-making, only a coarse-grained first approximation. Sense-making should fold into Orientation, and make it possible to expand your repertoire of Actions. That means synthesizing collected information into new knowledge. This is something linking supports, and tags do not. So if we have to choose one primitive, Wikilinks are the better primitive.

## Appendix 2: Terminology

### Specification requirement levels

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).


## Appendix 3: Background

### Similarities to Ted Nelson's ELF

By an accident of convergent evolution, Subtext happens to have some structural resemblances to Ted Nelson's ELF format (Nelson, 1965).

TODO elaborate.

### Background motivation

Subtext evolved out personal experiments with plain-text note-taking, spanning 10 years and 10k notes. Many of these notes were written in Markdown. However, with time, I noticed that the markup needs of note-taking are different from those of publishing. My note-taking style converged on a small subset of Markdown's features: text, links, lists, quotes, and one level of heading. To have more may be useful for publishing, but is often overkill for note-taking.

At the same time, I began to write small programs to work with this collection of notes, and started to run into limitations with Markdown, HTML and other formats. Parsing Markdown or HTML into an AST is possible, but in practice, is often a headache. Once you do have it parsed, it is often unclear how to meaningfully decompose or merge documents. Heading levels may need to be changed, lists may need to be flattened or nested. Because the document format is complex, foreknowledge of the meaning of the document is necessary to make meaningful programmatic changes.

A linear block-oriented format resolves the problem by radically simplifying it. With a linear data model, the range of meaningful document structures is narrowed, and this means you can make complex, yet meaningful programmatic decisions, without as much context about the specific document:

- Excerpt a document by taking the first text block
- Select all quotes from a collection of documents
- Select all links, and generate a graph for a collection of documents
- Find all backlinks and append them to the document as links

Linear, block-oriented formats are like shipping containers for discrete thoughts. Because they are structurally uniform, they can be automatically moved around and reorganized.

### Design notes

- [Hypertext Montage](https://subconscious.substack.com/p/hypertext-montage)
- [Thought Legos](https://subconscious.substack.com/p/thought-legos)

## References

- Nelson, Theodore "A File Structure for the Complex, the Changing, and the Indeterminate", Association for Computing Machinery, Proceedings of the 20th National Conference, 84-100. Ed. Lewis Winner, 1965.