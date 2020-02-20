# RFC Name

Request for Contribution (RFC) 0017: _Umbraco media item tracking_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: 

* Umbraco developers
* Umbraco users

## Summary

We would like to enable Umbraco to natively track references between items so that users can visually see how items are interlinked and where they are used. For example, we want to allow users to be able to see where a particular media item is being referenced, or for a developer to see what content is using a particular Macro.

For this RFC we will focus on Media tracking.

The goal of this RFC is to detail the MVP (Minimum Viable Product) for an initial release of Media tracking in the CMS and shed light on any issues or pitfalls with the approach described below. In the future, we will add further support for tracking Umbraco item relations.

## Motivation

There is currently no way to tell if an Umbraco item is being referenced. This makes cleanup a difficult process since deleting an item that is referenced can have an unintentional impact on content and potentially break functionality on a site. Adding tracking and overview of Umbraco item usage would help increase confidence when working in the backoffice as users have a chance to either avoid deleting an item or remove/change referenced items before deleting, mitigating unexpected outcomes. It will enable users and developers to be proactive instead of reactive when working with Umbraco items.

## Detailed Design

### Property Editors

The fundamental change in Umbraco to support this is to update the Property Editor APIs so that a Property Editor itself returns the UDIs that it is referencing. This will occur whenever an Umbraco content item (Content, Media, Member) is saved that uses Property Editors. When Umbraco saves the data, it will iterate over all of its property types and ask the corresponding Property Editor to return a list of UDIs. 

The method could be something along the lines of:

```cs
IEnumerable<Udi> GetReferences(TContent content, Property property);
```

This means the power of resolving related items is in the hands of the Property Editor since it knows exactly what data it is storing and how to process it.

The property editors that will need to be updated to use this method are:

* Content Picker
* Media Picker
* Member Picker
* MNTP
* Rich Text Editor
* Grid
* Multi URL Picker
* Nested Content
* Markdown editor

### Relations

When an Umbraco content item (Content, Media, Member) is saved and all of the UDI references have been resolved from the Property Editors, we will store these relations in the Umbraco Relations database table using the Umbraco Relations Service APIs. For content, we will be tracking relations for "Published" and "Pending" content.

The relations Service APIs may need to be enhanced to properly support Umbraco media item tracking and reporting detailed in this RFC.

_NOTE:_ We need to investigate how Umbraco Deploy is handling relations since we don't want to reinvent anything. We will either leverage what Umbraco Deploy is doing or Umbraco  Deploy will leverage this implementation.

### Media reporting

We intend on adding a report to the "Info" tab of a media item to show it's relations. The "Info" tab on a media item currently has quite a lot of available real estate so we plan on using that instead of creating another content app for this. 

This report will be a list showing which content, media and member items the media item is being used in. We will also indicate if these references are in the recycle bin. Each item will have a link to navigate to the related item and ideally, this will be in infinite editing but as an MVP this may mean linking to the item directly.

## Drawbacks

* Potential performance implications
* Tracked relations between Umbraco items might potentially become out of sync which can add more confusion

## Alternatives

Let users choose community packages to achieve this functionality such as Nexu. However, we want to be able to enable more than media tracking. We want to ensure that the functionality for reporting dependencies is baked into the Umbraco core editors instead of relying on external logic trying to manage these relations. We hope to borrow some inspiration from community packages and their users to achieve the best results.

## Out of Scope

* Although the design of this implementation will support tracking other Umbraco items other than media, the implementation of this RFC will focus solely on Media which means any reporting and functionality relating to other Umbraco items: Macros, Content, Templates, Members, Forms will not be part of the MVP.
* Users, Languages, Views, Partial Views, Stylesheets, Scripts will not be part of the relationship tracking APIs developed.
* For this MVP, reporting of media relations will be limited to the Info app on Media items only. Any additional reporting is outside of the scope of this RFC and may be included in future RFCs.
* For this MVP we do not plan on warning the user upon deletion of a media item if it is actively being used, in the future this will be a feature.
* For this MVP we do not plan on making it possible to create connectors/resolvers (similar to [Property Value Converters](https://our.umbraco.com/documentation/Extending/Property-Editors/value-converters)) for getting references from a property editor. We plan to add this functionality in the future.
* It will not be possible register a property editor for tracking via package.manifest in the MVP. We hope to add this functionality in the future. 

## Unresolved Issues

* How do we handle variants? The plan is to use the relations APIs and tables to do the tracking but they don't support "Culture" which would be required to link (for example) and media item to an English content variant. Currently, we can only link a media item to an entire Content item. To support this we would either need to: Change the relation table to have an additional language column - but this will mean that this table becomes less generic - and also change the relations APIs to support language, or we create a brand new tracking data set and services APIs. Still needs investigating.

## Related RFCs 

_No related RFCs_

## Contributors

This RFC was compiled by:

* Shannon Deminick (https://twitter.com/shazwazza)
* Rune Strand (https://twitter.com/hemraker)

Feedback, suggestions and comments contributed by:

* Nathan Woulfe
* Carole Rennie Logan
* Bjarne Fyrstenborg 
* Kenn Jacobsen
* Claus Jensen
* Dave Woestenborghs

Thanks for participating 🙌