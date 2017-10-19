# Annotations and instances: mapping W3C annotation model to BOA instances

Notes on mapping instances to annotations.

## Instances (name usages)
The [Australian National Species List](https://biodiversity.org.au/) uses “instances” to link taxonomic names to bibliographic references. An instance corresponds to a “usage” of a name, and looks like this https://biodiversity.org.au/nsl/docs/main.html#instance:

```
{
  "instance": {
    "class": "au.org.biodiversity.nsl.Instance",
    "_links": {...},        //links to this object and related resources
    "audit": {...},         //change information
    "namespace": "APNI",    //The shard or dataset this instance belongs to
    "verbatimNameString": "Doodia R.Br.", //The name string as written in the reference
    "page": "151",          //The page(s) of the reference this useage was found
    "pageQualifier": null,  //
    "nomenclaturalStatus": null,
    "bhlUrl": null,         //link to Biodiversity Heritage Librabry http://www.biodiversitylibrary.org/
    "instanceType": {},
    "name": {},
    "reference": {},
    "parent": {},           //Parent instance todo: explain
    "cites": {},            //An Instance that this instance cites
    "citedBy": {},          //An Instance that this instance is cited by
    "externalRefs": [],
    "instancesForCitedBy": [],
    "instancesForCites": [...],
    "instancesForParent": [],
    "instanceNotes": [...]
  }
}
```

I would prefer to use a more general model of annotation, so that we can treat name usages, name indexing (e.g., name finding in BHL OCR text), and human annotation (e.g., using a tool such as https://hypothes.is ) as essentially the same thing. In each case we have a string in some text that is being flagged as something important.

## The W3C annotation model 

The [W3C annotation model](https://www.w3.org/TR/annotation-vocab/) treats an annotation as comprising a **target** (the thing being annotated, such as a page in a paper) and a **body** (the annotation itself, e.g. the taxonomic name).

![image](https://raw.githubusercontent.com/rdmpage/annotations-and-instances/master/images/annotation.png)

## Fragment identifiers

Taxonomic name databases such as ION and IPNI often contain “microreferences” to the page in an article where a name first appears. Most web identifiers refer to documents (e.g., whole web pages, PDFs, XML files, etc.) rather than part of those documents. How can we link to parts of a digital document? In some cases documents parts may themselves be available as documents, for example, individual scanned pages in BHL each have a URL based on the PageID of that page. Hence we could refer to the page on which a new species name appears using the BHL PageID. For other documents an obvious approach is to use [fragment identifiers](https://en.wikipedia.org/wiki/Fragment_identifier) and https://www.w3.org/TR/annotation-model/#selectors 

### HTML

Use id of element containing the things being annotated.

### XML

We can use XPointer to identify locations in a XML document.  For example, #xpointer(//Rube)

```
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/anno22",
  "type": "Annotation",
  "body": "http://example.org/note1",
  "target": {
    "source": "http://example.org/page1.html",
    "selector": {
      "type": "XPathSelector",
      "value": "/html/body/p[2]/table/tr[2]/td[3]/span"
    }
  }
}
```

### PDF
 For a PDF we can use the **page** fragment identifier, see 
[RFC 3778 - The application/pdf Media Type](https://tools.ietf.org/html/rfc3778).
```
#page=<pagenum>
```

where **pagenum** is the ordinal page number starting at 1. Web browsers such as Chrome, and browser plugins such as Adobe Acrobat (https://helpx.adobe.com/acrobat/kb/link-html-pdf-page-acrobat.html) support this.
```
"selector": {
  "type": "FragmentSelector",
  "value": "page=10",
  "conformsTo": "http://tools.ietf.org/rfc/rfc3778"
}
```

## Mapping

Annotation | Instance | hypothes.is | remarks
--- | --- | --- | ---
target.source | bhlUrl | target.source | URI for the thing being annotated, e.g. web page, XML document, PDF, page image, etc.
target.scope | reference | uri | URI for the reference that includes the instance, e.g. DOI for the reference containing the name. Hypothes.is uses **urn:x-pdf** for PDFs, otherwise URI of resource being annotated
target.scope | . | document | hypothes.is also includes metadata about the article, e.g. from citation_* meta tags, as well as “documentFingerprint” (maybe a hash of the PDF?)
target.selector | page | target.selector | in BOA this is the page number, but we will use whichever fragment selector is appropriate for the source, e.g. XPath if it is XML
(value of text selector) | verbatimNameString | (value of text selector) | the name as it occurs in the text (instance), contents of annotation. If user has simply highlighted some text in hypothes.is then the name is likely to be in the text selector, will not be here unless they specifically add it.
body | name | | The body is the annotation itself. It can be text, other media, or a pointer to a resource. In this case it should be a URI for the name.
. | _links | links | links to this instance (BOA), links to annotation (API, web, in context) (hypothes.is)
id | _links | id | identifier for this annotation 
 Created, modified, etc. | audit | created, user | metadata about the annotation itself
motivation | | | Why are we creating the annotation? Two categories make sense here, “linking” and “identifying”. The first says “here’s a related resource”, the second says “this is   the thing”, so “identifying” and making body simply a link to a name URI seems a reasonable approach. However, “linking” may be safer as the annotation selector might not be to the level of the actual name string (for example, it might be a page, or a part of a page) in which case an assertion that the thing selected is a particular name may be too precise.

## Example

Below is an instance for [*Nuytsia floribunda* (Labill.) R.Br. ex G.Don](https://biodiversity.org.au/nsl/services/instance/apni/710750) formulated as an annotation of the BHL page [35116678](http://biodiversitylibrary.org/page/35116678)

```
{
	"@context": "http://www.w3.org/ns/anno.jsonld",
	"@id": "https://biodiversity.org.au/boa/instance/apni/710750",
	"created": "2011-09-08T14:00:00Z",
	"motivation": "linking",
	"body": "https://biodiversity.org.au/boa/name/apni/98247",
	"target": {
		"source": "http://biodiversitylibrary.org/page/35116678",
		"scope": "https://id.biodiversity.org.au/reference/apni/51640",
		"selector": {
			"type": "FragmentSelector",
			"value": "page=201",
			"conformsTo": "http://tools.ietf.org/rfc/rfc3778"
		}
	}
}
```

## What’s missing

![image](https://raw.githubusercontent.com/rdmpage/annotations-and-instances/master/images/synonymy-model-lego-style.png)





