# Python handler for ExifTool image metadata software

This python module provides a handler for
[ExifTool](https://exiftool.org/) software, which allows metadata
to be embedded within and extracted from image files. The most
important functionality added by the python module is

- to allow metadata to be embedded from templates that allow
  substitution fields

It can also be used:

- to extract the metadata fields to make them accessible to python software
- to display the metadata in a slightly nicer
  way than [ExifTool](https://exiftool.org/) allows natively

The python module has been written with application of the National
Centre for Atmospheric Science (NCAS) Image Metadata Standard in
mind. However, it may be used in any situation for which embedded
metadata fields need to be accessed.

The contents of this documentation page are as follows:

**Add navigation links here**

<a name="section_dependencies">

## Software dependencies

The python module depends on:

- [python 2.7](https://www.python.org/download/releases/2.7/) or
  [python 3.*](https://www.python.org/downloads/)
- a relevant version of [PyYAML](https://pypi.org/project/PyYAML/) for the
  version of python
* [ExifTool](https://exiftool.org/)

If the python module needs to used within a
[Cygwin](https://www.cygwin.com/) environment on a Windows computer,
[ExifTool](https://exiftool.org/) must be installed under Windows
rather than under [Cygwin](https://www.cygwin.com/).

<a name="section_nomenclature">

## Special nomenclature and concepts used in this documentation

Note that the following terms are local to this document rather than
being in common usage: inherent metadata field, optional metadata
field, short tag name, and full tag name.


<a name="nomenclature_inherent">
All digital image files contain **inherent metadata fields**,
e.g. for recording the image encoding system used and the size of the
image in terms of number of pixels. Although the values of these
fields can be extracted, they cannot be changed after the file has
been created.

<a name="nomenclature_optional">
Digital image files also have the capacity to have **optional metadata
fields** embedded within them. These may be embedded
at the time of file creation, as is typically the
case for photographs taken by digital cameras, or at any subsequent
time. It is possible to remove optional metadata fields or to change
their values.

<a name="nomenclature_tag_names"> 

**Tag names** are the handles used by
[ExifTool](https://exiftool.org/) to access the **(tag) values** of
metadata fields. Some of the **short tag names** given in the
[ExifTool documentation](#section_exiftool_man_pages) are ambiguous,
since they can be associated with more than one **family 1
group**. Consequently, this documentation makes use of **full tag
names**, which are colon-delimited concatenations of the family 1
group name and the short tag name. For example, the full tag name for
the Dublin Core *Creator* metadata field is *XMP-dc:Creator*. Note
that the tag names/descriptions displayed by metadata-aware image
viewing applications tend to be slightly different to the ones used by
ExifTool, but should be recognisably similar.

In general, **tag values** may be of Boolean, floating point, integer,
or string type. The ExifTool handler may extract all of these, but is
only designed to embed values of string type. Such values may include
**Unicode characters**, which is useful since this permits the use of
symbols such as for degrees "°".

<a name="nomeclature_controlled">

Some tag values are **controlled**, which means that they [must conform to a
specific format](#section_controlled_values). In most cases, this is because the values are
intended to be interpreted automatically by computer systems, e.g for
fields containing:

- [date-time information](#section_controlled_datetime_values)
- [Global Positioning System (GPS) coordinates](#section_controlled_gps_values)

<a name="nomenclature_list_tag">
**List tags** are metadata fields that may contain multiple
values. The order of these values is preserved when they are extracted from the
file. It is permissible for them to contain only a single value. 

<a name="nomeclature_structured_tag">

**Structured tags** may also contain multiple values, albeit as
closely-related key/value pairs and so are more like a python
dictionary than a list. In practice, [ExifTool](https://exiftool.org/)
allows each element to be accessed using a **flattened tag name**,
which makes it look like any other (non-list-tag) metadata field.

Metadata fields can be of 3 basic types, which are embedded in slighly
different ways. However, since they are all handled in the same way by
[ExifTool](https://exiftool.org/), it is not necessary to know
anything about the technical differences. 

- [Exchangeable image file format (officially known as
  Exif)](https://en.wikipedia.org/wiki/Exif) metadata fields are
  typically used by standalone digital cameras to record image capture
  settings. Smartphone cameras tend to make use of corresponding XMP
  metadata fields (see below) instead.
- [International Press Telecommunications Council (IPTC) Information
  Interchange Model
  (IIM)](https://en.wikipedia.org/wiki/IPTC_Information_Interchange_Model)
  fields are no longer commonly used, but might be found in older files. 
- [Extensible Metadata Platform
  (XMP)](https://en.wikipedia.org/wiki/Extensible_Metadata_Platform)
  metadata fields are probably the most-commonly-used type. All of the
  fields covered by the NCAS Image metadata standard are of the XMP
  type. The family 1 group names used by
  [ExifTool](https://exiftool.org/) are derived by appending the
  official namespace prefixes to a leading *XMP-*. Some group names
  are shortened for convenience.


<a name="section_usage_examples">

## Usage examples

All of the following usage examples should be initiated with one of
the two following code snippets, depending on whether they are being
run under a python 2.7 or python3.* environment:

````python
# For python 2.7
import module_exiftool_python2
handler = module_exiftool_python2.Handler()

# For python 3.*
import module_exiftool_python3
handler = module_exiftool_python3.Handler()
````

`<image-file-path>` will be used to represent the path of an image file. This
may be an absolute or a relative path and may use "`~`" to represent the path
of a home directory.

<a name="usage_display">

### Displaying embedded metadata

The metadata embedded within a file can be displayed using the
[display method](#method_display):

 ```` python
handler.display(<image-file-path>)
````

The [displayed metadata remain available to the ExifTool
handler](#section_extracted_metadata) until metadata are
displayed/extracted for a new file or until metadata are embedded
within a file. Consequently, the contents of a targetted file can be
redisplayed at any time using the same handler method, but without an
input argument:

 ````python
handler.display()
````

Moreover, the display can be redirected to a text file, whose path is
given `<output-file-path>`, using:

 ````python
handler.display(output_file_path=<output-file-path>)
````

Note that [the contents of the output file will be in exactly the same
format as shown on the screen](#section_extracted_metadata). This has
been optimised for human rather than machine readability and is not
intended to serve as a storage format.

<a name="usage_extract">

### Extracting embedded metadata

The metadata embedded within a file can be extracted, i.e. made
accessible to a python computer program, using the [extract
method](#method_extract):

 ````python
metadata = handler.extract(<image-file-path>)
````

where [`metadata` is a python dictionary](#section_extracted_metadata)
whose values are accessed using [full tag
names](#nomenclature_tag_names) as keys.

As for the case of displaying metadata, the metadata remain available to the
handler and can be subsequently displayed using the [display method](#method_display) with no input argument.

<a name="usage_load_a_template">

### Loading a template

A [template](#section_template_files), whose file path is given by
`<template-file-path>`, will need to be loaded before any of the
subsequent uses of the ExifTool handler can be demonstrated. The
accompanying example template file,
`module_exiftool_example_template.yaml`, can be used for this
purpose. The template is loaded using the [load_a_template
method](#method_load_a_template):

 ````python
template_id = handler.load_a_template(<template-file-path>)
````

where `template_id` is a string that uniquely identifies the template
(in the context of the other templates loaded by the handler). Its
value is defined within the file.

The ids of all templates that have been loaded by the handler can be seen
using the [show_templates_available method](#method_show_templates_available).

 ````python
handler.show_templates_available()
````

<a name="usage_test_from_template">

### Testing emebedding metadata from a template

The [test_from_template method](#method_test_from_template) is
intended for testing how metadata will be embedded within a file
without actually embedding the metadata. It takes 1, 2, or 3 input
arguments, which progressively test different aspects of the embedding
process. The input arguments are:

- `template_id`, i.e. the string that uniquely identifies the template (in the
  context of other templates loaded by the handler). The value is defined
  within the template file and is returned by the [load_a_template
  method](#usage_load_a_template). 

- `substitutions` is a python dictionary containing the specific
  values that must be substituted into the
  [template](#section_template_files). The handler will select the
  values by the keys that match the substitution field names used in
  the relevant template file. Note that the `substitutions` dictionary
  may contain entries in addition to those required for the template
  in question. If the template contains no substitution fields, an
  empty python dictionary, i.e. `{}`, should be used when supplying
  the [test_from_template method](#method_test_from_template) with 2
  or 3 input arguments.

- `<image-file-path>` is the path of the image file into which the metadata
  are intended to be embedded. 

Supplying the [test_from_template method](#method_test_from_template) with just
the first argument:

 ````python
handler.test_from_template(template_id)
````

will:

- allow you to check that any [short tag
  names](#nomenclature_tag_names) supplied within the template have
  been interpreted as belonging to the intended [family 1
  group](#nomenclature_tag_names)

- highlight substitution fields with bold text, if your terminal
  allows this. An alternative way of displaying the substitution
  fields within a template is to use the [show_template_requirements
  method](#method_show_template_requirements).

Note that the handler will automatically add metadata fields
*XMP-xmp:MetadataDate* and *XMP-xmp:ModifyDate*, whose values will
represent the current Coordinated Universal Time (UTC)
date-time. Consequently, entries for these fields should not be added
to the template.

Supplying the [test_from_template method](#method_test_from_template) with
the first two input arguments:

 ````python
handler.test_from_template(template_id, substitutions)
````

will test whether the substitution values have been formatted as
expected. These will be highlighted with bold text if your terminal will allow
it. If the template contains no substitution fields, an empty python
dictionary, i.e. `{}`, should be supplied.

The following would be appropriate for the example template file:

 ````python
import datetime, math
substitutions = {
    "string_substitution": "aardvark"
    "float_substitution": math.pi}
````

Note that it is not necessary to provide a substitution value for
<em>\_\_utcnow\_\_</em>, which is automatically available and gives the current
UTC date-time.

Supplying the [test_from_template method](#method_test_from_template) with
all three input arguments:

 ````python
handler.test_from_template(template_id, substitutions, <image-file-path>)
````

will test whether any metadata fields would be overwritten by embedding the
new metadata. 


<a name="usage_embed_from_template">

### Embedding metadata from a template

**WARNING**: Information can be lost if you overwrite an existing
metadata field. You should make use of the [test_from_template
method](#usage_test_from_template) until you are confident that any
overwrites are deliberate.

The order of input arguments required by the [embed_from_template
method](#method_embed_from_template) is deliberately the same as that
used for [testing how metadata would be
embedded](#usage_test_from_template):

 ````python
exit_code = handler.embed_from_template(template_id, substitutions, <image-file-path>)
````

where 

- `template_id`, i.e. the string that uniquely identifies the template (in the
  context of other templates loaded by the handler). The value is defined
  within the template file and is returned by the [load_a_template
  method](#usage_load_a_template). 

- `substitutions` is a python dictionary containing the specific
  values that must be substituted into the
  [template](#section_template_files). The handler will select the
  values by the keys that match the substitution field names used in
  the relevant template file. Note that the `substitutions` dictionary
  may contain entries in addition to those required for the template
  in question. If the template contains no substitution fields, an
  empty python dictionary, i.e. `{}`, should be supplied.

- `<image-file-path>` is the path of the image file into which the metadata
  are intended to be embedded. 

- `exit_code` is the value returned by the method: 0 if the metadata were
  embedded successfully or 1 if an error was encountered


<a name="section_handler_methods">

## Handler methods

This section gives details of each of the handler's methods. 

<a name="section_usage_examples">

<dl>
  <a name="method_display">
  <dt><b>display</b>(<em>option="extracted"[, output_file_path=None]</em>)</dt>
  <dd>Displays the metadata extracted from a file. It is essentially an
  alternative to running the [ExifTool](https://exiftool.org/) command
  `exiftool -G1 -args <image-file-path>`, albeit with some advantages.

  - If the path of an image file is supplied as the first input argument,
    *option*, the method will display the metadata extracted from that file.
  - If no input argument is given, the method will display the metadata
    extracted from the the last image file targetted. Note that these metadata
    become unavailable after new metadata are embedded into a file (since the
    embedded metadata will have changed).
  - Supplying the path of a text file as the second input argument,
    *output_file_path*, will cause the display to be outputted to that
    file. The directory part of the file path must correspond to an existing
    directory. The display format has been optimised for human
    rather than machine readability and is not intended to serve as a storage
    format.
  - Note that [not all of the displayed metadata fields are extracted from the
  file](#section_extracted_metadata). 

  Metadata are shown in 3 columns: 

  - the 1st shows the [full tag name](#nomenclature_tag_names). Fields are
    arranged with their full tag names in alphabetical order (there is no
    inherent order to extracted metadata fields).
  - the 2nd indicates whether the value is interpreted as a string (s),
    integer (i), float (f), or boolean (b) data type. Note that most values
    will be of string type, but might be interpreted as integers or floats by
    the extraction. Consequently, no assumptions should be made about the data
    type when using the [extract method](#method_extract).
  - the 3rd shows the value, which will be wrapped onto the next line
    if it is too long to fit on the first. Otherwise, line breaks are
    only shown where a *\\n* has been included in the value.
 
  In the case of a [list tag](#nomenclature_list_tag) that contains multiple
  values, each value will be shown as a separate entry with an incremental
  number appended to the [full tag name](#nomenclature_tag_names) within
  square brackets.

  The display can be redirected to a text file by supplying a suitable file
  path as the second (optional) input argument (*output_file_path*). Note that
  the display format is optimised for human rather than machine
  readability. The display will not be redirected if the directory part of the
  supplied file path does not exist.

  [Refer to the fine details section to find out more about extracted
  metadata](#section_extracted_metadata).


  <a name="method_embed_from_input">
  <dt><b>embed_from_input</b>(<em>input_metadata, image_file_path</em>)</dt>
  <dd>Embeds metadata from the supplied input into a file.

  This method is intended as an alternative to the
  [embed_from_template method](#method_embed_from_template) for simple
  situations when only a few metadata fields are needed. However, 
  there is no limit on the number of fields that may be supplied. This method does not permit substitution fields.

  The first input argument (*input_metadata*) should be a python dictionary:

  - whose keys are either [full tag names](#nomenclature_tag_names) or
    unambiguous [short tag names](#nomenclature_tag_names)
  - whose values are the values that are intended to be embedded within the
    file

  The second input argument (*image_file_path*) should be the path of
  the file into which the metadata fields will be embedded.

  An exit code of 0 is returned by the method to indicate that
  metadata were embedded successfully, whereas an exit code of 1 is
  returned to indicate that they were not.
  
  For example, the following could be used:

  ````python
  input_metadata = {
      "XMP-dc:Creator": "Hooper, David A.",
      "XMP-photoshop:Headline": "This is a trvial example"}
  exit_code = handler.embed_from_input(input_metadata, <image-file-path>)
  ````

  The [test_from_input](#method_test_from_input) method can
  alternatively be used to test the implications of embedding the
  metadata from the input dictionary, without actually doing so.


  <a name="method_embed_from_template">
  <dt><b>embed_from_template</b>(<em>template_id, substitutions,
    image_file_path</em>)</dt>

  <dd>Embeds metadata from a [template](#section_template_files),
  which may contain substitution fields, into a file.

  This first input argument (*template_id*) provides the identifier of
  a template that has previously been loaded using the
  [load_a_template](#method_load_a_template) method.  

  The second input argument (*substitutions*) must be a python dictionary
  whose values will be substituted into the chosen template at locations
  identified by its keys. The dictionary may contain key-value pairs in
  addition to the ones required for the particular template. However, no
  attempt will be made to embed the metadata if the dictionary does not
  contain entries for all of the substitution fields required by the
  template. An empty python dictionary must be supplied in the case of a
  template that contains no substitution fields.

  The third input argument (*image_file_path*) provides the path of
  the file into which the metadata should be embedded.

  An exit code of 0 is returned to indicate that metadata were
  embedded successfully, whereas an exit code of 1 is returned to
  indicate that they were not.

  The [test_from_template](#method_test_from_template) method can
  alternatively be used to test the implications of embedding the metadata
  from a template within the file, but without actually doing so.

  The [embed_from_input](#method_embed_from_input) method can alternatively be
  used for simple situations, e.g. when only a few metadata fields are
  required to be embedded.


  <a name="method_extract">
  <dt><b>extract</b>(<em>image_file_path</em>)</dt>
  <dd>Returns a python dictionary containing the metadata extracted from a file.

  The single input argument (*image_file_path*) gives the path of the
  file from which metadata are to be extracted.

  An empty python dictionary will be returned if any errors were
  encountered whilst trying to extract metadata.

  Refer to the [Extracted metadata section](#section_extracted_metadata) for
  more details about the python dictionary returned by this method. 

  The metadata will remain avaiable to the handler until either the
  [display](#method_display) method is used to display the metadata for a new
  file or new metadata are embedded within a file using the
  [embed_from_input](#method_embed_from_input) method or the
  [embed_from_template](#method_embed_from_template) method. Consequently,
  they may be displayed using the [display](#method_display) method with no
  input argument.


  <a name="method_load_a_template">
  <dt><b>load_a_template</b>(<em>template_file_path</em>)</dt>
  <dd>Loads a metadata template.

  The single input argument *template_file_path* should give the path
  of template file.

  This method will return a value of an identifying string *template_id*,
  which is defined within the template file. This value is subsequently needed
  by the handler in order to use either the
  [test_from_template](#method_test_from_template) method or the
  [embed_from_template](#method_embed_from_template) method. An empty string
  will be returned if any error is encountered when loading the template.

  <a name="method_return_datetime_string">
  <dt><b>return_datetime_string</b>(<em>supplied_datetime</em>)</dt>
  <dd>Returns a string representation of a datetime object in a format suitable
    for embedding within a file.

  The supplied datetime object is assumed to represent UTC. By default, this
  method will return a string in *%Y:%m:%d %H:%M:%S* (python) format. The
  letter *Z* can be appended to this (in order to explicitly indicate UTC) by
  supplying the value *Z* for option *timezone_indicator* using the
  [set_option](#method_set_option) method. The default option is to omit the
  letter *Z*.


  <a name="method_set_option">
  <dt><b>set_option</b>(<em>option_name, option_value</em>)</dt>
  <dd>Allows the values of handler options to be set.

  The names, values, and permissible values for the options available to the
  handler can be displayed using the [show_options](#method_show_options)
  method.

  The first input argument (*option_name*) should provide the name of the
  option and the second input argument whould supply the required value
  *option_value*.

  A value of 0 will be returned if the option was set successfully, but a
  value of 1 will be returned if it was not. 

  <a name="method_show_options">
  <dt><b>show_options</b>()</dt>
  <dd>

  Shows the names, values, and permissible values for options
  available to the handler.

  The first column shows the option name. The second column shows the
  current value. The third column shows the permissible values. 

  Note that in the case of options whose permissble values are *True*
  and *False*, these will be shown as *1* or *0*, respectively, in the
  second column.

  The values of these options may be changed using the
  [set_option](#method_set_option) method.


  <a name="method_show_recognised_tags">
  <dt><b>show_recognised_tags</b>()</dt>
  <dd>Shows the [full tag names](#nomenclature_tag_names) of metadata fields
  recognised by the handler.

  A letter *A* in the first column indicates that the corresponding [short tag
  name](#nomenclature_tag_names) is ambiguous, since it may be associated with
  more than one metadata group. Such metadata fields may only be accessed (by
  the [embed_from_template](#method_embed_from_template) and
  [embed_from_input](#method_embed_from_input) methods) using the [full tag
  name](#nomenclature_tag_names), which is highlighted. All other metadata
  fields may be accessed using either the [short tag
  name](#nomenclature_tag_names) (which is highlighted) or the [full tag name](#nomenclature_tag_names).

  The symbol *+* in the second column indicates that the corresponding
  metadata field is a [list tag](#nomenclature_list_tag).

  <a name="method_show_template_requirements">
  <dt><b>show_template_requirements</b>(<em>template_id</em>)</dt>
  <dd>Shows the substitution fields contained within a template.

  The same information is shown, albeit in a different way, by using the
  [test_from_template](#method_test_from_template) method with a single input
  argument.

  The single input argument (*template_id*) should be the value of the
  identifying string that was returned by the
  [load_a_template](#method_load_a_template] method when the template was
  loaded. 


  <a name="method_show_templates_available">
  <dt><b>show_templates_available</b>()</dt>
  <dd>Shows the values of *template_id* for all templates that have been
  loaded using the [load_a_template](#method_load_a_template) method.

  <a name="method_test_from_input">
  <dt><b>test_from_input</b>(<em>option="extracted"[, output_file_path=None]</em>)</dt>

  <a name="method_test_from_template">
  <dt><b>test_from_template</b>(<em>template_id[, substitutions[,
  file_path]])</em>)</dt>
  <dd>Tests how metadata will be embedded from a template into a file, but
  without actually doing so.

  The tests carried out depend on whether one, two, or all three input
  arguments are supplied.

  The first input argument should supply the id (*template_id*) of one of the
  templates that have been loaded using the
  [load_a_template](#method_load_a_template) method. If only the first input
  argument is supplied, this method will:

  - indicate the [full tag names](#nomenclature_tag_names) assumed 
    for any [short tag names](#nomenclature_tag_names) used in the
    template
  - highlight the location of any substitution fields within the template. The
    names of these substitution fields can alternatively be shown using the
    [show_template_requirements](#method_show_template_requirements) method.

  Note that the handler will automatically add fields
  ````XMP-xmp:MetadataDate```` and ````XMP-xmp:ModifyDate````, unless the
  value of option *add_standard_tags:* is set to *False* using the
  [set_option](#method_set_option) method. The handler automatically provides
  access to substitution fields ````__utcnow__```` (a datetime object that
  represents the current UTC date-time) and ````__utcnowstr__```` (a string
  that represents the value of the current UTC date-time as returned by the
  [return_datetime_string](#method_return_datetime_string) method) without
  them needing to be provided by the second input argument. 

  The second input argument (*substitutions*) should be a python dictionary
  containing the names and values of susbtitution fields within the
  template. If only the first two input arguments are supplied, the method
  will highlight the locations of the substitution values. If the template
  does not contain any substitution fields, an empty dictionary, i.e. *{}*,
  should be supplied.

  The third input argument (*file_path*) should give the path of the file into
  which the metadata are intended to be embedded. If all 3 input arguments are
  supplied, the method will indicate the names of any metadata fields that
  would be overwritten.


  <a name="method_test_from_template">
  <dt><b>test_from_template</b>(<em>template_id[, substitutions[, file_path]]</em>)</dt>
  <dd>Tests embedding metadata from a template into a file without comitting the changes.

  This method is designed to be analogous to the one
  ([embed_from_template](#mmethod_embed_from_template)) used to embed
  metadata from a template. It allows each stage of the process to be
  tested, by providing 1, 2, or 3 input arguments.
  
  This first input argument (template_id) provides the identifier of a
  template that has previously been loaded using the
  [load_a_template](#method_load_a_template) method. If only the first
  input argument is provided, the template will be displayed with:

  <ul>
    <li>any template errors indicated
    <li>any unrecognised tags marked with a **U** in the first column
    <li>any substitution fields highlighted in bold (for terminals that allow this)
  </ul>





The level of
  testing depends on how many input arguments are provided.


  If only the first input argument (*template_id*) is provided, 


provides the identifier of
  a template that has previously been loaded using the
  [load_a_template](#method_load_a_template) method. 

  The second input argument (*substitutions*) must be a python
  dictionary whose values will be substituted into the chosen template
  at locations identified by its keys. The dictionary may contain
  key-value pairs in addition to the ones required. However, no
  attempt will be made to embed the metadata if the dictionary does
  not contain all of the required substitution fields. An empty python
  dictionary may be supplied in the case of a template that contains
  no substitution fields.

  The third input argument (*file_path*) should provide the path of
  the file into which the metadata should be embedded.

  An exit code of 0 is returned to indicate that metadata were embedded
  successfully and and an exit code of 1 to indicate that they were not.
  


</dl>


## Fine details

<a name="section_extracted_metadata">

### Extracted metadata

The [extract method](#method_extract) and [display method](#method_display)
rely on the [ExifTool](https://exiftool.org/) command:

````
exiftool -G1 -j -c %+.6f <image-file-path>
````

- The `-G1` option ensures that [full tag names](#nomenclature_tag_names) are
  returned rather than [short tag names](#nomenclature_tag_names)
- The `-j` option returns the metadata in [JavaScript Object Notation (JSON)
  format](https://en.wikipedia.org/wiki/JSON), which can be handled by python
  software
- the `-c %+.6f` option is the handler's default option for the [format in
  which GPS latitudes and longitudes are
  extracted](#section_controlled_gps_values). This can be changed.

The metadata are extracted as a python dictionary (available as
`handler.metadata["extracted"]` and also returned by the [extract
method](#method_extract)), whose keys:

- (mostly) represent the [full tag names](#nomenclature_tag_names) of the
  metadata fields
- access the values of those metadata fields

Note that no assumptions should be made about the data type of the values
returned by ExifTool/the handler, since:

- In the case of [list tags](#nomenclature_list_tag) that contain multiple
  values, the dictionary keys access python lists. In all other cases,
  including for [list tags](#nomenclature_list_tag) that contain a single
  value, the dictionary keys access a single, non-list value.
- Some string type values can be interpreted by ExifTool as being either an
  integer or float type. For example, this affects negative (but not positive)
  values of [GPS latitudes and longitudes](#section_controlled_gps_values). It
  is suspected that this is result of the [JavaScript Object Notation (JSON)
  encoding](https://en.wikipedia.org/wiki/JSON). The [display
  method](#method_display) indicates the interpreted data type by the letter
  *"b"* to indicate a Boolean value, *"f"* to indicate a float, *"i"*, and
  *"s"* to indiate a string.
  
Not all of the metadata fields provided within the python dictionary are
extracted from the file:

- the key *SourceFile* accesses the value of `<image-file-path>`
  provided to [ExifTool](https://exiftool.org/). Note that this is the only
  key that does not conform to the format of a [full tag
  name](#nomenclature_tag_names), i.e. by containing a colon (which separates
  a [group name](#nomenclature_tag_names) from a [short tag
  name](#nomenclature_tag_names)). This field is ignored by the [display
  method](#method_display).
- the key *ExifTool:ExifToolVersion* accesses the version of
  [ExifTool](https://exiftool.org/) used used to extract the metadata
- keys with group name *System* are derived from the operating system
- keys with group name *Composite* are derived by
  [ExifTool](https://exiftool.org/) from information already available in both
  the [inherent metadata fields](#nomenclature_inherent) and [optional
  metadata fields](#nomenclature_optional). They do not contain any additional
  information and are only provided for convenience. The number of such fields
  depends on how much information is contained within the embedded metadata
  fields.

Two other points to note:

- Since the metadata are extracted into a python dictionary, they have no
  inherent order. The [display method](#method_display) shows them with the
  keys (and hence the [full tag name](#nomenclature_tag_names)) in
  alphabetical order.
- the values of [matadata fields containing date-time and GPS coordinate
  values](#section_controlled_values) will be given in bespoke formats, which
  differ from those used to embed the values.


<a name="section_template_files">

### Template files

Template files make use of [YAML syntax](https://yaml.org/spec/1.2.2/), which
is designed to be both human and machine readable. Only a small subset of the
syntax is used and so it should not be necessary to refer to the [full YAML
documentation](https://yaml.org/spec/1.2.2/). The accompanying
example file `module_exiftool_example_template.yaml` should be used to
identify all of the features described below.

- Identation is significant in [YAML syntax](https://yaml.org/spec/1.2.2/) and
  so all lines should begin no indentation (except in the case of multi-line
  values, which will be described shortly).

- Comments may be included, either in-line or on separate lines, following a
  hash symbol "#". For a multi-line comment, each line must begin with the
  hash symbol "#".

- Each entry should be given in *"- key: value"* format, i.e. the line should
  begin with a dash "-", followed by a space, followed by the *key*, followed
  by a colon ":", followed by a space, followed by the *value*.

- One entry should use the key *template_id* to define the value of the
  template id, i.e. a string that uniquely identifies the template in the
  context of other templates loaded by the handler. The value of *template_id*
  is returned by the [load_a_template method](#method_load_a_template) when a
  template is loaded from a file. It is useful to give this as the first entry
  in the file, but it is not obligatory to do so.

- For all other entries, the *key* should be either a [short tag
  name](#nomenclature_tag_names) or [full tag
  name](#nomenclature_tag_names) and *value* represents the value that should
  be embedded within that metadata field. Note that the only [recognised tag
  names](#section_recognised_tags) may be used.

- Note that the values of fields that are designed to contain [date-time or
  GPS coordinate information must be supplied using controlled
  formats](#section_controlled_values).

- For a value that is intended to contain line breaks, the entry should begin
  with *"- key: |"*, where the "|" character indicates that *value* will begin
  on the next line. The first line of *value* should be indented by more than
  the first charcter of *key* on the previous line, i.e. by at least 4 spaces
  (although the exact number of spaces is arbitrary). All subsequent lines of
  *value* should begin with at least the same level of indentation as the
  first line. If more indentation is used, the extra spaces are interpreted as
  being part of *value*. For example, this can be used to indicate the
  beginning of all paragraphs other than the first one. Line breaks within
  *value* will be included in the embedded value.

- Substitution fields may be included in *value* using [python format string
  syntax](https://docs.python.org/3/library/string.html#format-string-syntax).
  The general syntax is `{substitution_field_name:format_specification}`,
  i.e. with the substitution field name and the required format specification
  separated by a colon ":" and enclosed within curly braces ("{" and "}"). For
  example, the syntax {substitution_field_name:.2f} may be used to 

If
  default string formatting is required, which is likely to be true in most
  cases, the syntax may be simplified to `{substitution_field_name}`. 


  For
  example, the syntax `{supplied_datetime:%Y:%m:%d %H:%M:%S]` may be used
  to provide the value for a field 

represent a substution field named `supplied_datetime` whose value should
  be formatted in a way appropriate to 

a date-time substitution value.  If *value* is intended to contain a curly
  brace, i.e. either "{" or "}", as a character rather than as an indication
  subsitution field, it should be escaped by using the charcter twice,
  i.e. "{{" or "}}". If *value* begins with a substitution field, the whole of
  *value* must be conatined within either single or double quotation marks
  (otherwise the YAML parser will interpret *value* incorrectly).

- There are two ways to assign multiple values to a [list
  tag](#nomenclature_list_tag). If the values may all fit on one line, the
  snytax `- tag_name: [value_1, value_2, value_3]` may be used, i.e. with the
  values separated from each other by a comma and a space ", " and all
  enclosed within square brakets, "[" and "]". Otherwise, each value may be
  given on a separate line using the syntax
  ````python
    - tag_name:
      - value_1
      - value_2
      - value_3
  ````

- The order in which entries are given is arbitrary.


<a name="section_recognised_tags">

### Recognised tags

By default, the handler will only allow values to be embedded within metadata
fields that are "recognised", i.e. if the corresponding [full tag
name](#nomemclature_tag_names) is given within the accompanying file
*module_exiftool_recognised_tags.dat*. The file also contains an indication of
whether the metadata field is a list tag (by prefixing the tag name with a "+"
symbol). Details of how to add additional tag names to this file are given at
the bottom of this section.

This file is required in order to overcome a few peculiarities of ExifTool:

- Some [short tag names](#nomemclature_tag_names) are "ambiguous" in that they
  can be used to represent two or more [full tag
  names](#nomemclature_tag_names) that are associated with different metadata
  groups, e.g. *ExifIFD:CreateDate* and *XMP-xmp:CreateDate*. ExifTool will
  decide which [full tag name](#nomemclature_tag_names) to use depending on
  the context, although this choice might not be what was
  intended. Consequently, tyhe handler will only accept [short tag
  names](#nomemclature_tag_names) that are "unambiguous", i.e. that are
  associated with only one metadata group in the recognised tags file. Details
  of the recognised tags can be shown using the [show_recognised_tags
  method](#method_show_recognised_tags).
- ExifTool will give an error message if an invalid tag name (i.e. one that it
  rather than the handler recognises) is supplied. However, it go ahead and
  embed the values into all other metadata fields for which valid tag names
  have been supplied. The handler is more conservative and will not allow any
  values to be embedded unless all supplied tag names are recognised.
- If an attempt is made to embed multiple values within a metadata field that
  is not a [list tag](#nomenclature_list_tag), ExifTool will not give an error
  message and will simply embed the final supplied value into the
  corresponding metadata field. The handler will not accept multiple values
  for any metadata field that is not indicated as being a [list
  tag](#nomenclature_list_tag) by the recognised tag file.

If you need to embed values within a metadata field that is not covered by the
recognised tags file, you will need to consult the [ExifTool man
pages](#section_exiftool_man_pages) in order to determine:

- the appropriate [full tag name](#nomemclature_tag_names), by appending the
  the [short tag name](#nomemclature_tag_names) to the metadata group name,
  separated by a colon ":" (without adding spaces).
- whether or not the metadata field is a [list tag](#nomenclature_list_tag)

The format of the recognised tags file is very simple

- Any lines begininning with the "#" symbol can be used to give comments. They
  will be ignored by the handler.
- The [full tag name](#nomemclature_tag_names) should be preceeded by the "+"
  symbol if it corresponds to a [list tag](#nomenclature_list_tag). 
- In order to aid human readability, spaces have been used in the recognised
  tags file to indent the [full tag names](#nomemclature_tag_names) of non
  [list tags](#nomenclature_list_tag) and to separate the "+" symbol from the
  the [full tag name](#nomemclature_tag_names) in the case of [list
  tags](#nomenclature_list_tag). However, these are not necessary. Moreover,
  aribitrary numbers of spaces may be used.
 
In principle, the handler can be forced to accept unrecognised tag names by
supplying the option name *allow_unrecognised_tags* and option value *False*
to the [set_option method](#method_set_option). However, this will allow
preventable errors to go undetected. Consequently, it is much better to make
use of the recognised tags file.  

<a name="section_exiftool_man_pages">

### ExifTool man pages

On a Unix-like operating system, including Linux, the "man" page (i.e. the
native documentation) covering ExifTool can be accessed using the command:

````
man exiftool
````

The man page covering tag names can be accessed using the command:

````
man Image::ExifTool::TagNames
````

Since the documentation is rather long, particularly in the case of that
covering tag names, the following commands: 

| Action | Command |
| :---  | :--- |
|Show full list of commands | < h > |
|Go forwards by 1 line | < Return > |
|                      | <↓> |
|Go backwards by 1 line | <↑> |
|Go forwards by 1 page | < Space > |
|   	              | < Page Down > |
|Go backwards by 1 page | < b > |
|Search forwards for first instance of *pattern* | < / >*pattern*< Return > |
|Search backwards for first instance of *pattern* | < ? >*pattern*< Return > |
|Search forwards for next instance of last-used search *pattern* | < n > |
|Search backwards for next instance of last-used search *pattern* | < N > |
|Return to the top of the man page | < g > |


<a name="section_controlled_values">

### Metadata field that require controlled values

Values for metadata fields that contain 

- date-time information
- Global Positioning System (GPS) coordinates

are stored within files in controlled
formats. [ExifTool](https://exiftool.org/) will accept the values in a wider
variety of formats and will convert them before embedding them within a
file. Moreover, it will convert the controlled values into a preferred format
when extracting them from a file. 

<a name="section_controlled_datetime_values">

#### Metadata fields that contain date-time information

In the case of XMP family metadata fields that are designed to contain
date-time information, e.g.:

  - XMP-dc:Date
  - XMP-xmp:CreateDate
  - XMP-xmp:MetadataDate
  - XMP-xmp:ModifyDate

the recommended format for supplying values to be
embedded by ExifTool is the following, which is the format used when the
values are extracted:

*YYYY:mm:dd HH:MM:SS[.ss][TZD]*

where

- *YYYY* is the 4 digit year
- *mm* is a two digit representation of the month (e.g. *01* for January and
  *12* for December)
- *dd* is a two digit representation of the day of the month (i.e. between *01*
  and *31*)
- *HH* is a two digit representation of the hour (i.e. between *00*
  and *23*)
- *MM* is a two digit representation of the minute (i.e. between *01*
  and *59*)
- *SS* is a two digit representation of the seconds (i.e. between *01*
  and *59*)
- *ss* is one or more digits representing a decimal fraction of a second
- *TZD* is a time zone designator, either *Z* to represent Coordinated
  Universal Time (UTC), or *+hh:mm*/*-hh:mm* to represent the offset from UTC.

Note that:

- the decimal fraction of a second (*ss*) and time zone designator (*TZD*)
  elements are optional. Neither may be used for Exif family metadata fields
  that are designed to contain date-time information, (
  e.g. *ExifIFD:CreateDate*, *ExifIFD:DateTimeOriginal*, and
  *IFD0:ModifyDate*), which are commonly used by stand alone cameras to record
  the image capture date-time. There are separate metadata fields for these
  pieces of information.
- if the time zone designator (*TZD*) is not included, the time zone is assumed
  to be unknown. Note that it is omitted for the NCAS Image Metadata standard.
- when embedding metadata, ExifTool will allow virtually any character to be
  supplied as a delimiter between the date-time elements, e.g. with a dash "-"
  rather than a colon ":" between the date elements and with a "T" between the
  date and time elements. However, when extracting metadata, the values will
  always be returned using the format shown above.
- lower levels of date-time granularity may be used, e.g. to represent just a
  date, using a [subset of ISO 8601
  representations](http://www.w3.org/TR/Note-datetime.html) formats.

The handler provides the [return_datetime_string
method](#method_return_datetime_string), which returns a string in *YYYY:mm:dd
HH:MM:SS* format by default, e.g.:

````python
import datetime
datetime_string = handler.return_datetime_string(datetime.datetime.utcnow())
````

The letter "Z" may be added in order to represent
UTC by supplying the option name `"timezone_indicator"` and the option value
`"Z"` to the [set_option method](#method_set_option):

````python
handler.set_option("timezone_indicator", "Z")
datetime_string = handler.return_datetime_string(datetime.datetime.utcnow())
````

The method has no option to represent an offset from UTC. 

An alternative is to use the [python format string
  syntax](https://docs.python.org/3/library/string.html#format-string-syntax)
  *%Y:%m:%d %H:%M:%S* in a [template substitution
  field](#section_template_files).

<a name="section_controlled_gps_values">

#### Metadata fields that contain Global Positioning System (GPS) coordinates

A bit more care needs to be taken with GPS latitudes and longitudes since

- irrespective of the format of values supplied to ExifTool to embed them
  within a file (any of the options shown in the table below may be used),
  they are converted into a bespoke format, which appears to include fields
  for degrees (*D*), minutes (*M*), and seconds (*S*)
- ExifTool provides no option to extract the values in their native format and a
  conversion format must be specified (or the default option will be used)

The handler allows the extraction format to be specified by supplying the
option name `gps_extraction` with one of the values of `option_value` from the
table below to the [set_option method](#method_set_option). The default option
value is *+D*, since this is the most convenient format for handling by a
computer program. The Latitude and Logitude columns in the table below show
how the same values will be extracted using the different extraction format
options.

  option_value | Extraction format   | Latitude        | Longitude
  :---   | :---          | :---            | :---
  *+D*   | %+.6f         |+52.424500       |-4.005467
  *D*    | %.6f          |52.424500 N      |4.005467 W
  *+DM*  | %+d %+.4f     |+52 +25.4700     |-4 -0.3280
  *DM*   | %d %.4f       |52 25.4700 N     |4 0.3280 W
  *+DMS* | %+d %+d %+.2f |+52 +25 +28.20   |-4 -0 -19.68
  *DMS*  | %d %d %.2f    |52 25 28.20 N    |4 0 19.68 W

The extraction conversion format may include

- just decimal degrees (*D*)
- integer degrees and decimal minutes (*DM*) 
- integer degrees, integer minutes, and decimal seconds (*DMS*) 

The extracted values may be:

- "signed", in which case the `option_value` begins with a "+" symbol and
  positive/negative values are used to indicate latitudes to the north/south
  of the equator and longitudes to the east/west of the primary meridian
- "unsigned", in which case case the letters *N/S* are used to indicate
  latitudes to the north/south of the equator and the letters *E/W* to
  indicate longitudes to the east/west of the primary meridian. The letter is
  separated from the final decimal value by a single white space.

Note that:

- ExifTool's default extraction format is a variation on the handler's *DMS*
  option. This results in the values of latitude and longitude from the table
  above being extracted as *52 deg 25' 28.20" N* and *4 deg 0' 19.68" W*,
  respectively.

- the numbers of decimal places used by the handler for extraction have been
  chosen to provide the same level of precision as used by the typical GPS
  receiver *DM* representation, which uses 4 decimal places for the minutes. The
  same levels of precision should be used when providing the handler with
  values to embed. 

- If a "signed" extraction option is specified, ExifTool interprets negative
  (but not positive) values of latitude and longitude as floating point
  numbers rather than as strings. It is not clear why this happens, but could
  be a result of the JSON encoding.

- care should be taken when using the "signed" *+DM* and *+DMS* format for
  embedding values. Each element of the value must be prefixed by a "-" sign,
  not just the first (degrees) one. Otherwise, subsequent elements will be
  interpreted as positive perturbations from the negative degrees value,
  e.g. *-4 0 19.68* will be interpreted as *3 59 40.32 W*. When extracting
  values, the handler will also include a "-" sign before each element of the
  value. Positive values do not need to be prefixed with a "+" sign.


GPS altitudes are treated differently:

- they are understood to be given in units of metres
- when embedding values, the should be provided to the handler as string that
  represents just an integer, i.e. without any symbol to indicate the units
- when extracting values, ExifTool will automatically add the symbol "m"
  (after a single space) to represent units of metres. This cannot be changed.


### ExifTool gotchas

ExifTool has a few peculiarities, which can be confusing.

When displaying metadata using variations on the command `exiftool
<image-file-path>`:

- string values than contain instances of "\n" in order to represent
  line breaks are not shown split across lines. Instead, the instances
  of "\n" are replaced by a full stop ".". This can make the output
  difficult to read. The handler shows the values with the
  line breaks included.

- for [list tags](#nomeclature_tag_names) that contain multiple
  values, ExifTool will show the values concatenated with a comma
  followed by a space, i.e. ", ", as a delimiter. It is not possible
  to distinguish this from a single value that just so happens to
  contain commas. The handler shows each value on a separate line with
  an incremental number shown in quare brackets after the [full tag
  name](#nomenclature_tag_names).


End of documentaion 



