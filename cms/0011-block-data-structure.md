# RFC Name

Request for Contribution (RFC) 0019 : Block-based data structure interoperability (Project Proteus, version 2)

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: 
* Technical users
* Developers 
* Package Developers

## Summary

We would like to define a standard for storing structured data as a foundation for complex Umbraco editors, such as Grid Layouts / DocType Grid Editor (DTGE) / LeBlender Editor, Nested / Stacked / Inner Content editors, etc.

This project has the codename [Proteus](https://en.wikipedia.org/wiki/Proteus), named after the Greek god who was "capable to assume many forms"; and that is exactly what we want to achieve with this RFC.

**The short term goals are:**
- Standardise the way we utilise Umbraco elements (ElementTypes)
- Provide a standard for separating content from presentation data within complex editors
- Having a core framework for block manipulation to eventually implement packages like Grid Layouts, Nested Content or other `PublishedElement` oriented Property Editors (out-of-scope for this RFC)
- Make sure that all the content is structured, easy to manipulate, predictable and indexed properly.

**The long term goals are:**
- Make the content of elements shareable, stored directly in the database as opposed to in JSON blobs. (out-of-scope for this RFC)
- Use variants and segmentation (out-of-scope for this RFC)

## Motivation

Currently, there are many complex editors, both in Core and as community packages, that store complex data structures in their different ways. This makes the data more difficult to consume and re-use. Furthermore, design or implementation specific configuration is often mixed with the data, which restricts the flexibility and reusability of some editors (e.g. Grid Layouts).

The varying implementations result in equally varying developer experiences when utilising these editors. Having a common foundation for data storage means we can share common functionality across many parts of the CMS, and secure a more uniform developer experience.

The lack of consistency makes it challenging or impossible for data manipulation, including: 

- Indexing the information
- No way to reuse the data

## General Design

The available "content blocks" on a complex editor is configured in a Prevalue Editor specific to the Property Editor.
We would be using Element Types to define the structure of each "content block".

## Detailed Design

We propose creating a uniform way to store the data of a complex editor. The data will consist of two main parts:

**Layout:** The Layout will define Blocks that each will reference a content item located in the list of `data` by it's UDI. The Layout object will be key/value pairs where the `key` is the Property Editor alias.

**Data:** A list of content items based on ElementTypes(`IPublishedElement`).


#### Simple example of JSON

```json
{
  "layout": {
    "Umbraco.NestedContent": [
      {
        "udi": "umb://element/fffba547615b4e9ab4ab2a7674845bc9",
        "settings": {}
      }, 
      {
        "udi": "umb://element/e7dba547615b4e9ab4ab2a7674845bc9",
        "settings": {}
      }
    ]
  },
  "data": [
    {
      "udi": "umb://element/fffba547615b4e9ab4ab2a7674845bc9",
      "title": "Hello"
    },
    {
      "udi": "umb://element/e7dba547615b4e9ab4ab2a7674845bc9",
      "title": "World",
      "text": "The content of a textarea"
    }
  ]
}

```

### Layout
The Layout will contain an object with a structure specific to the Property Editor, named by the Property Editor Alias.
This is required because some editors will not store a simple one-dimensional array such as Nested Content, but rather multi-dimensional structures like Grid Layouts. By defining each layout with the alias of the Property Editor it means developers can in theory swap the underlying Property Editor without losing data and while keeping the layout preserved by the previous editor.

The structure of a given layout object is defined by the Property Editor and can therefore vary depending on the Property Editor.

Common for all is that the object referencing content items would implement the `IBlockElement` interface. This interface has two properties:

* UDI — Used to link to the content item.
* settings — An object containing properties needed for a specific layout. The setting could contain the number of columns for ie. Grid Layouts. Settings are not required and can therefore be empty. It would be posible for developers to change or extend the settings. Potentially a Property Editor could provide the option to pick a specific Element Type (`IPublishedElement`) for settings for each Block.

### Data
A list of Element Type based content items (`IPublishedElement`)
  - Element Types are Document Types without any routable settings (e.g. templates, URL).
  - In the short term the element data will be stored as JSON and each element will have it's own UDI assigned.


### Strongly typed

* The content items of `data` are strongly typed as `IPublishedElement`.
* The layout can be strongly typed too but will be up to the Property Editor to define.

We do have a specific interface for the object representing a Block item, notice this can be extended by the specific Property Editor. The `IBlockElement` C# interface would look like this:

```cs

public interface IBlockElement<TSettings>
{
    Udi Udi { get; }
    TSettings Settings { get; }
}
```

This also means these can be strongly typed and used by Models Builder to generate models.

### Future

In the future `data` could be stored in the database as a document rather than embedded JSON objects. This unlocks the possibility for variant content, segmentation and scheduled publishing blocks by leveraging similar behaviour already in the Core CMS. (This part is out of the scope of this RFC).

## Drawbacks

Compatibility between existing complex editors (Grid, Nested / Stacked / Inner Content, DTGE, etc) and the proposed model will unfortunately not be possible.

Due to this, block based implementations of existing complex editors might need to be maintained alongside the existing editors for a while. This could cause confusion for developers about which editors to be using, however the old ones can be marked as deprecated so they no longer show up in the backoffice for new projects.

An upgrade or migration path between specific editors could be defined at a later stage.

## Out of Scope

- Creating new implementations of existing complex editors as block-based editors - separate RFCs will be proposed to determine the exact approaches per editor
- Creating a mechanism to create and maintain PublishedElements as distinct documents in the database (Ability to share block content between editors)
- Extending behaviour of `IPublishedElement`, including permissions and scheduled publishing
- Use variants and segmentation at element level (out-of-scope for this RFC)
- Validation of complex editors - We have the capability for this already but more prototypes and docs need to be written

## Contributors

This RFC was based on previous RFC on the same topic: [RFC — 0011 — block-data-structure](https://github.com/umbraco/rfcs/pull/11/files?short_path=d8a15c1#diff-d8a15c17ff96b16ae9d652b0cbc3b6ca) compiled by:
- [Callum Whyte](https://twitter.com/callumbwhyte) (community)
- [Nathan Woulfe](https://twitter.com/nathanwoulfe) (community)
- [Kenn Jacobsen](https://twitter.com/KennJacobsen_DK) (community)
- [Jeffrey Schoemaker](https://twitter.com/jschoemaker1984) (community)
- [Antoine Giraud](https://twitter.com/aaantoinee) (community)
- [Niels Hartvig](https://twitter.com/thechiefunicorn) (HQ)
- [Shannon Deminick](https://twitter.com/shazwazza) (HQ)

This updated RFC is compiled by:
- [Niels Lyngsø](https://twitter.com/nielslyngsoe) (HQ)
- [Claus Jensen](https://twitter.com/clausjnsn) (HQ)
- [Shannon Deminick](https://twitter.com/shazwazza) (HQ)
- [Rune Strand](Strand) (HQ)

