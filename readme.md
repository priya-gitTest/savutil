# savutil - converting SPSS .sav file data to open formats

The sav2util utilities convert an SPSS� .sav file to Triple-S XML metadata and
data with an intermediary JSON format.</p>

savutil is based on the Windows DLL provided by IBM for programmed access to SAV files.

savutil comprises two programs:

* sav2json - converts a SAV file into an intermediate JSON format
* json2sss - converts a JSON file created by sav2json into Triple-S data set

<p>sav2sss produces up to three output files. They are named after the input
file, with different extensions:</p>
<ul>
  <li>XML schema file: <em>sav-file-name</em>.xml</li>
  <li>Data file: <em>sav-file-name</em>.asc (fixed-format mode) or .csv (CSV
    mode)</li>
  <li>Documentation file: <em>sav-file-name</em>.txt - <em>only if the .sav
    file contains documentation text</em></li>
</ul>

## Installation on Windows

Download the two executables from this github project into a folder of your own
choice, let's say <some-folder>. Both programs must be executed from a command
prompt.

## Running sav2json

### The sav2json command line

```
<some-folder>\sav2json [switches] <SAV-file>
```

where <SAV-file> is the path to and name of the SAV file to be converted.

#### Switches taking no value:

* The -c switch if specified forces output of a CSV file
* The -d switch if specified includes the data in the JSON file as well as
  the descriptions of the variables. This is mandatory if json2sss is to be run
  afterwards.
* The -h switch if specified includes a header line in the CSV file
* The -i switch if specified causes values in the CSV file if generated to
  be replaced by their SPSS value labels if available
* The -p switch if specified causes the JSON output to be "pretty-printed" for
  readable. By default the JSON is compact.
* The -t switch if specified causes any descriptive text in the SAV file to be written
  to a text file - 
* The -v switch if specified displays the sav2json version number.

#### Switches requiring a value.

NB: values are terminated by the next space or
hyphen. <em>To include a space or hypen in a value, enclose the switch and
value in double quotes, e.g. "-tMy title"</em>

<ul>
  <li>The -e switch specifies the encoding to be used in the CSV file if generated,
    by default cp-1252 (which should be fine for Windows
    users in almost all locales). The JSON file is always encoded in UTF-8 as this
    is part of the JSON specification.</li>
  <li>The -o switch specifies the path and root file name for the output files. By default
  the output files (.json, .csv, .txt) have the same path and name as the SAV file.</li>
</ul>
<ul>
  <li>The -s switch disables "sensible string lengths". This is a default
    option useful for data sets coming from Quancept � which may have extremely
    long string variable lengths. By default each long string variable is
    reduced to a length no greater than the next power of 2 greater than the
    maximum size found in the file for that variable.
  <li>The -x switch can be used to specify the value of the 'user' element in
    the XML file. The value should not contain a semicolon (';') character.
    Note that switches whose values contain spaces should be enclosed in double
    quotes in the command line as shown above. The user element does not appear
    by default. </li>
  <li>The -h switch is used to specify an href attribute for the &lt;record&gt;
    element. By default sav2sss includes an href which is a relative reference
    to the .asc file. To exclude the href altogether use
    <pre>"-h "</pre>
  </li>
  <li>The -t switch is used to specify contents of either of the &lt;name&gt;
    and &lt;title&gt; elements in the XML. Separate the name and title by a
    semicolon. There is no default name or title. If there is no semicolon the
    whole value is used as the title. E.g.
    <pre>"-tOmnibus201401:January 2014 Omnibus survey"</pre>
    <p>Name is 'Omnibus201401' and title is 'January 2014 Omnibus survey' </p>
    <pre>"-tJanuary 2014 Omnibus survey"</pre>
    <p>No name, and title is 'January 2014 Omnibus survey' </p>
  </li>
  <li>The -d switch is used to trigger interpreted CSV mode (see below) and
    specify a delimiter for multiple values in the output file.</li>
  <li>The -e switch is used for identifying multiple variables recorded in the .sav file as consecutive single variables. These are typically identified with a common prefix, a delimiter such as underscore, and then a suffix reflecting the identity of each category. This suffix is usually an integer corresponding to the position of the category in the answer list, i.e. _1 for the first answer, _2 for the second answer and so on. For this case, specify "-e_" to assemble (for example) a multiple Q1 from SPSS variables Q1_1, Q1_2 etcetera. Sometimes the suffices are not of this form, typically if the suffices are alphabetic or integers not consecutive and starting from one. To specify suffices individually, add them after the delimiter character separated by colons, e.g. "-e@A:B:C:D:E:F:G:H" to specify that the delimiter is at-sign and the suffices are A, B, C, D, E, F, G and H.<p>The default is "-e" i.e. sav2sss will not attempt to infer and create multiple variables from the SPSS data.
  </p>
  </li>

## Example

```
sav2json -dp survey
```

Converts the file survey.sav in the current working directory creating
the file survey.json, including the data values and formatting the JSON for
readability.

</ul>

<h2>Metadata</h2>

<p>The contents of the &lt;user&gt; element may be controlled by the -x
parameter as described above. Otherwise:</p>
<ul>
  <li>The &lt;date&gt; element shows the date of the SAV2SSS run</li>
  <li>The &lt;time&gt; element shows the time of day of the SAV2SSS run</li>
  <li>The &lt;origin&gt; element contains
    <pre>SAV2SSS {version} (Windows) by Computable Functions (http://www.computable-functions.com)</pre>
  </li>
</ul>

<h2>Character encodings</h2>

<p>Sav files have the ability to store text in several different encodings
internally. This should be retrieved correctly by sav2sss but it has not been
tested with a wide variety of character sets. Any encoding provided in the <a
href="http://docs.python.org/2/library/codecs.html#standard-encodings">Python
standard library</a> may be used as the data encoding. Characters from the sav
file that are not representable in the specified data encoding are rendered in
the data file as ?. The XML is always written in encoding ISO-8859-1, so all
Unicode characters are representable, and the XML file should be easily viewed
and edited on Windows machines. The data file is written by default with
encoding CP1252; this is a superset of ISO-8859-1 that is almost universally
recognised.</p>

<h2>Missing values</h2>

<p>A .sav file may declare certain values for a variable to be missing values.
Triple-S represents missing values with a blank field, so sav2sss outputs all
missing values as blanks. The output generated by the -f switch shows which
values have been treated in this way. The codes for missing values are not
included in the XML file (these codes never appear in the data).</p>

<h2>Anomalous code values</h2>

<p>SPSS allows all variables including numerics to have codes for missing
values. It also allows categorical variables to be incompletely coded, and have
negative code values. This is not compatible with the requirements for
categorical variables in Triple-S. Therefore variables are only treated as
categorical, i.e. single or multiple, if all the values appearing in the file
have been coded, and only valid codes (zero or higher integer) are used, except
for missing values which may have any code because they are not written to the
output file. A variable must have at least two categories after excluding
missing values and invalid codes in order to be treated as categorical.
Variables rejected on the basis of these criteria are converted as Triple-S
quantity variables with a range based on the width declared for them in
SPSS.</p>

<h2>Multiple categorical variables</h2>

<p>These variables are represented in .sav files by a sequence of consecutive
regular variables, identifiable as follows:</p>
<ul>
  <li>there is one variable per category of the multiple, in the sequence of
    the categories</li>
  <li>each variable has the same root name with an underscore followed by an
    integer as a suffix (with some complications where adding the suffix makes
    the variable name into a long name) OR (as described above for the -e command option) a delimiter
    followed by a suffix drawn from a list, e.g. the letters A, B, C etcetera.</li>
</ul>

<p>Together with one of the following scenarios:</p>

<h4>Bitstring representation</h4>
<ul>
  <li>each variable has a title comprised of the underlying multiple variable
    title with the answer label of the corresponding category</li>
  <li>each variable has an answer list containing either:
    <ul>
      <li>exactly two answers coded as zero (answer not applicable) or one
        (answer applicable)</li>
      <li>identical answer lists containing a specific label for Yes and
        another for No (as specified by the -y and -n switches)</li>
    </ul>
  </li>
</ul>

<p>The resulting Triple-S multiple has regular format, with one field per
category containing zero or one.</p>

<p>sav2sss recognises this scenario and exports it in the XML as a Triple-S
multiple variable with the bitstring representation. The data file
representation is identical to that of the sequence of single variables.</p>

<h4>Spread representation</h4>

<p>This representation is used for sparse multiples, where the list of
alternatives may be long but only a few selections are expected.</p>
<ul>
  <li>Each variable has a label comprised of the underlying multiple variable
    title with with the standard answer (from the -m switch) corresponding to
    its position in the sequence of components. I.e. using the defaults, the
    first variable label will contain "1st answer", the second "2nd answer" and
    so on.</li>
  <li>Each variable has an identical answer list</li>
</ul>

<p>The resulting Triple-S multiple has 'spread' format, with one field per
component variable.</p>

<h4>Prefixes/suffixes</h4>

<p>The variation in component variable labels may come at the beginning of the
label, i.e. the labels have a fixed suffix which is the underlying question
label, or at the end, i.e. the labels use the question label as a fixed prefix
with the category label at the end. The prefix or suffix is separated from the
varying part by a delimiter, typically a colon (:). You may specify either a
prefix or a suffix, but not presently both, using the -a or -b switches. The
default is to look for an unvarying prefix separated from the varying suffix by
a colon as delimiter. The delimiter is not included in the XML, and may be
longer than one character, e.g. "-b: ".</p>

<h4>Consistency</h4>

<p>sav2sss assumes that coding conventions will be consistent in any one .sav
file, i.e.:</p>
<ul>
  <li>where the Yes/No representation of multiples is used, the same answer
    labels are always used for Yes and No.</li>
  <li>where spread representation is used, the same labels are always used for
    the first answer, second answer etcetera.</li>
</ul>

<h2>CSV output</h2>

<p>If the -c switch is used a .CSV file is generated as specified in the
Triple-S standard. Note that all multiple values are enclosed in quotes, not
just those whose first character is zero. The motivation is consistent
treatment if the CSV file is read into Excel, every value in each column will
be treated as either numeric or character and aligned accordingly.</p>

<h4>Interpreted output</h4>

<p>The switch -d may be used to created interpreted output in the .csv file,
i.e. each column for a categorical variable will contain the variable label and
not its code. The character specified as delimiter (e.g. -d\ to specify
backslash) is used to separate the values of a multiple variable within its
column. The CSV file generated in this way <strong>is not Triple-S
compliant</strong>. However, this output can be very useful with analysis
applications that have no support for metadata, e.g. Excel pivot tables. NB:
the -d switch is required to invoke this mode even if there are no multiple
variables in the file. In this mode it does not usually make sense to enable the
features of sav2sss that create Triple-S multiple variables.</p>

<h2>Known Issues</h2>
<ol>
  <li>Values marked as being in time/date format in the sav file are not
    translated. The timestamp values in the files seem to be well outside the
    Unix epoch. So time/date format values are translated as missing values.
    NB: character strings recording time and/or date are not affected; only
    variables explicitly defined as time/date values.</li>
  <li>This may be an instance of a more general problem that some
    floating-point values in .sav files seem to be eleven times larger than one
    would expect, depending on the context in the sav file. sav2sss tries to
    take account of this context but without understanding of the motivation
    this is clearly perilous.</li>
  <li>Numeric fields are assigned digits and decimal places based on a
    combination of the information about formats in the sav file and the actual
    values found. </li>
</ol>

<h2>Using the source (for Python developers)</h2>

savutil was developed with Python 2.7.

<h4>Prequisites</h4>

sav2json uses the file savdllwrapper.py which was developed by  
<h4>Testing</h4>

<p>To run sav2sss in the Python interpreter
run the scripts sav2json.py and json2sss.py</p>

<h4>Building</h4>
<ol>
  <li>Download the sources.</li>
  <li>Building the Windows executable requires the <a
    href="http://www.py2exe.org/">py2exe library</a>. </li>
  <li><strong>Review the script setup.bat for its suitability on your
    system</strong>.</li>
  <li>Execute setup.bat to create sav2json.exe, json2sss.exe and readme.html in a subfolder
    .\output</li>
</ol>
</body>
</html>
