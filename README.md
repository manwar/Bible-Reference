# NAME

Bible::Reference - Simple Bible reference parser, tester, and canonicalizer

# VERSION

version 1.01

[![Build Status](https://travis-ci.org/gryphonshafer/Bible-Reference.svg)](https://travis-ci.org/gryphonshafer/Bible-Reference)
[![Coverage Status](https://coveralls.io/repos/gryphonshafer/Bible-Reference/badge.png)](https://coveralls.io/r/gryphonshafer/Bible-Reference)

# SYNOPSIS

    use Bible::Reference;

    my $r = Bible::Reference->new;
    $r = Bible::Reference->new(
        bible    => 'Protestant', # or "Orthodox" or "Catholic" or "Vulgate"
        acronyms => 0,            # or 1
        sorting  => 1,            # or 0 to preserve input order
    );

    $r = $r->in('Text with I Pet 3:16 and Rom 12:13-14,17 references in it.');

    my $refs = $r->refs;
    # 'Romans 12:13-14, 17; 1 Peter 3:16'

    my $books = $r->as_books;
    # [ 'Romans 12:13-14, 17', '1 Peter 3:16' ]

    my $verses = $r->as_verses;
    # [ 'Romans 12:13', 'Romans 12:14', 'Romans 12:17', '1 Peter 3:16' ]

    my $hash = $r->as_hash;
    # { 'Romans' => { 12 => [ 13, 14, 17 ] }, '1 Peter' => { 3 => [16] } }

    my $array = $r->as_array;
    # [[ 'Romans', [[ 12, [ 13, 14, 17 ]]]], [ '1 Peter', [[ 3, [16] ]]]]

    my $text = $r->as_text;
    # 'Text with 1 Peter 3:16 and Romans 12:13-14, 17 references in it.'

    $r = $r->in('More text with Romans 12:16, 13:14-15 in it.'); # appends "in"
    $r = $r->clear; # clears "in" data but not anything else

    my @books  = $r->books;
    my @sorted = $r->sort( 'Romans', 'James 1:5', 'Romans 5' );

    $r->bible('Vulgate');        # switch to the Vulgate Bible
    $r->acronyms(1);             # output acronyms instead of full book names
    $r->sorting(0);              # deactivate sorting of references
    $r->require_verse_match(1);  # require verses in references for matching
    $r->require_book_ucfirst(1); # require book names to be ucfirst for matching

# DESCRIPTION

This module is intended to address Bible reference canonicalization. Given some
input, the module will search for Bible references, canonicalize them, and
return them in various forms desired. It can return the canonicalized within
the context of the input string or strings as well.

The module supports the Protestant Bible by default and by configuration
setting also the Orthodox Bible, the current Catholic Bible, and the Vulgate.

There are also some potentially useful helper methods.

# METHODS

## new

A typical instantiation method that accepts some settings, all of which can
later be fetched and changed with accessors.

    my $r = Bible::Reference->new(
        bible    => 'Protestant', # or "Orthodox" or "Catholic" or "Vulgate"
        acronyms => 0,            # or 1
        sorting  => 1,            # or 0 to preserve input order
    );

See the below accessor methods for details on these settings.

## bible

This accessor method gets and sets the current Bible to use. By default, the
Bible is the Protestant Bible (since this is most common). Other Bibles
supported are the Orthodox, current Catholic, and Vulgate Bibles.

You can set the value to any substring of the name.

    $r->bible('c'); # sets Bible to "Catholic"

## acronyms

This accessor method gets and sets the boolean setting of whether to return
full book names (which is the default) or acronyms.

    $r->acronyms(0);         # default
    $r->in('Rom 1:1')->refs; # returns "Romans 1:1"

    $r->acronyms(1);
    $r->in('Rom 1:1')->refs; # returns "Ro 1:1"

## sorting

This accessor method gets and sets the boolean setting of whether or not to
return references sorted (which is the default) or in their input order.

    $r->sorting(1);                   # default
    $r->in('Jam 1:1; Rom 1:1')->refs; # returns "Romans 1:1; James 1:1"

    $r->sorting(0);
    $r->in('Jam 1:1; Rom 1:1')->refs; # returns "James 1:1; Romans 1:1"

Note that within a single given reference, chapters and verses will always be
returned sorted and canonicalized.

## in

This method accepts string input that will get parsed and canonicalized by the
module. The method returns a reference to the object.

    $r = $r->in('Text with I Pet 3:16 and Rom 12:13-14,17 references in it.');

The method is also additive, in that if you call it multiple times or with a
list of input strings, the object stores them all (until you call `clear`).

    $r->in('Text with I Pet 3:16 and Rom 12:13-14,17 references in it.');
    $r->in('More text with Roms 12:16, 13:14-15 in it.');
    $r->in(
        'Even more text with Jam 1:5 in it.',
        'And one last bit of text with 1 Cor 12:8-12 in it.',
    );

## clear

This method clears all input provided via `in` and returns a reference to the
object.

    $r = $r->clear; # clears "in" data but not anything else

## refs

This method returns all references found within the input. It does so as a
single string using canonical reference format.

    my $refs = $r->refs;
    # 'Romans 12:13-14, 17; 1 Peter 3:16'

The "canonical reference format" is as follows: Book names are proper-noun cased
followed by a single space. If the book name has a number prefix, it is in
numeric form followed by a single space. Chapter numbers are next, followed by a
":" and verses. For uninterrupted ranges of verses, the "-" character is used.
Interrupted verses (i.e. "14, 17") are displayed with a comma and space between
them.

Multiple book references are separated by a ";" and space. Multiple chapters
within the same book are separated by a space and comma: "Romans 12:14, 17:18; 1
Peter 3:16" Therefore, whole chapters that follow chapters with verses will
repeat of the book name (for disambiguation). Whole chapters that only follow
whole chapters will not repeat the book name.

## as\_books

This method is the same as `refs` except that it returns a list or arrayref
(depending on context) of canonicalized references by book.

    my $books = $r->as_books;
    # [ 'Romans 12:13-14, 17', '1 Peter 3:16' ]

    my @books = $r->as_books;
    # 'Romans 12:13-14, 17', '1 Peter 3:16'

## as\_verses

This method is the same as `as_books` except that it returns a list or arrayref
of independent verses,

    my $verses = $r->as_verses;
    # [ 'Romans 12:13', 'Romans 12:14', 'Romans 12:17', '1 Peter 3:16' ]

    my @verses = $r->as_verses;
    # 'Romans 12:13', 'Romans 12:14', 'Romans 12:17', '1 Peter 3:16'

## as\_hash

This method returns the references output like `refs` would except that the
output is a hash or hashref (depending on context) of a tree of data.

    my $hash = $r->as_hash;
    # { 'Romans' => { 12 => [ 13, 14, 17 ] }, '1 Peter' => { 3 => [16] } }

    my %hash = $r->as_hash;
    # 'Romans' => { 12 => [ 13, 14, 17 ] }, '1 Peter' => { 3 => [16] }

## as\_array

This method is the same as `as_hash` except that the output is an array or
arrayref (depending on context) of a tree of data.

    my $array = $r->as_array;
    # [[ 'Romans', [[ 12, [ 13, 14, 17 ]]]], [ '1 Peter', [[ 3, [16] ]]]]

    my @array = $r->as_array;
    # [ 'Romans', [[ 12, [ 13, 14, 17 ]]]], [ '1 Peter', [[ 3, [16] ]]]

## as\_text

This method returns a text string or, if there were multiple calls to `in`, an
array or arrayref of text strings (depending on context), of the input string or
strings with the references found therein canonicalized.

    my $text = $r->as_text;
    # 'Text with 1 Peter 3:16 and Romans 12:13-14, 17 references in it.'

    $r->clear;
    $r->in('Text with I Pet 3:16 and Rom 12:13-14,17 references in it.');
    $r->in('More text with Roms 12:16, 13:14-15 in it.');
    $r->in(
        'Even more text with Jam 1:5 in it.',
        'And one last bit of text with 1 Cor 12:8-12 in it.',
    );

    my @text = $r->as_text;
    # 'Text with 1 Peter 3:16 and Romans 12:13-14, 17 references in it.',
    # 'More text with Romans 12:16, 13:14-15 in it.',
    # 'Even more text with James 1:5 in it.',
    # 'And one last bit of text with 1 Corinthians 12:8-12 in it.',

## books

This method returns a list or arrayref (depending on the context) of books of
the Bible, in order.

    my @books = $r->books;
    my $books = $r->books;

# HANDLING MATCHING ERRORS

By default, the module does its best to find things that look like valid
references inside text. However, this can result in the occational matching
error. For example, consider the following text input:

    This is an example of the 1 time it might break.
    It also breaks if you mention number 7 from a list of things.
    Legal opinions of judges 3 times said this would break.

With this, we'd falsely match: Thessalonians 1, Numbers 7, and Judges 3.

There are a couple things you can do to reduce this problem. You can optionally
set `require_verse_match` to a true value. This will cause the matching
algorithm to only work on reference patterns that contain what look to be
verses.

You can optionally set `require_book_ucfirst` to a true value. This will cause
the matching algorithm to only work on reference patterns that contain what
looks like a book that starts with a capital letter (instead of the default of
any case).

# SEE ALSO

You can look for additional information at:

- [GitHub](https://github.com/gryphonshafer/Bible-Reference)
- [CPAN](http://search.cpan.org/dist/Bible-Reference)
- [MetaCPAN](https://metacpan.org/pod/Bible::Reference)
- [AnnoCPAN](http://annocpan.org/dist/Bible-Reference)
- [Travis CI](https://travis-ci.org/gryphonshafer/Bible-Reference)
- [Coveralls](https://coveralls.io/r/gryphonshafer/Bible-Reference)
- [CPANTS](http://cpants.cpanauthors.org/dist/Bible-Reference)
- [CPAN Testers](http://www.cpantesters.org/distro/B/Bible-Reference.html)

# AUTHOR

Gryphon Shafer <gryphon@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2017 by Gryphon Shafer.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
