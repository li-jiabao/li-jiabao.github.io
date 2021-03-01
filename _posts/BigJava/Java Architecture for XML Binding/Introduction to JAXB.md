
# JAXB Architecture

This section describes the components and the interactions in the JAXB processing model.

## Architectural Overview

The following figure shows the components that make up a JAXB implementation.

Figure:&#160; JAXB Architectural Overview

A JAXB implementation consists of the following architectural
components:

- Schema compiler: Binds a source schema to a set of schema-derived program elements. The binding is described by an XML-based binding language.
- Schema generator: Maps a set of existing program elements to a derived schema. The mapping is described by program annotations.
- Binding runtime framework: Provides unmarshalling (reading) and marshalling (writing) operations for accessing, manipulating, and validating XML content using either schema-derived or existing program elements.

## The JAXB Binding Process

The following figure shows what occurs during the JAXB binding process.


Figure:&#160;Steps in the JAXB Binding Process

The general steps in the JAXB data binding process are:

1. Generate classes: An XML schema is used as input to the JAXB binding compiler to generate JAXB classes based on that schema.

1. Compile classes: All of the generated classes, source files, and application code must be compiled.

<li>Unmarshal: XML documents written according to the constraints in the source schema are unmarshalled by the JAXB binding framework. Note that JAXB also supports unmarshalling XML data from sources other than files and documents, such as DOM nodes, string buffers, SAX sources, and so forth.
</li>
1. Generate content tree: The unmarshalling process generates a content tree of data objects instantiated from the generated JAXB classes; this content tree represents the structure and content of the source XML documents.
1. Validate (optional): The unmarshalling process involves validation of the source XML documents before generating the content tree. Note that if you modify the content tree in Step 6, you can also use the JAXB Validate operation to validate the changes before marshalling the content back to an XML document.
1. Process content: The client application can modify the XML data represented by the Java content tree by using interfaces generated by the binding compiler.
1. Marshal: The processed content tree is marshalled out to one or more XML output documents. The content may be validated before marshalling.

## More About Unmarshalling

Unmarshalling provides a client application the ability to convert XML data into JAXB-derived Java objects.

## More About Marshalling

Marshalling provides a client application the ability to convert a JAXB-derived Java object tree into XML data.

By default, the <tt>Marshaller</tt> uses UTF-8 encoding when generating XML data.

Client applications are not required to validate the Java content
tree before marshalling. There is also no requirement that the Java
content tree be valid with respect to its original schema to marshal
it into XML data.

## <a name="bnazn" id="bnazn"></a>More About Validation

Validation is the process of verifying that an XML document meets
all the constraints expressed in the schema. JAXB 1.0 provided
validation at unmarshal time and also enabled on-demand validation on
a JAXB content tree. JAXB 2.0 only allows validation at unmarshal and
marshal time. A web service processing model is to be lax in reading
in data and strict on writing it out. To meet that model, validation was added to marshal time so users could confirm that they did not invalidate 
an XML document when modifying the document in JAXB form.