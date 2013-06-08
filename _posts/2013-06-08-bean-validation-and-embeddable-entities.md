---
layout: post
category : programming
tags : [Bean Validation, JPA]
author: Fabian Stäber
---
{% include JB/setup %}

This post shows how to use [JSR-303](http://jcp.org/en/jsr/detail?id=303)
Bean Validation on `@Embeddable` [JPA](http://jcp.org/en/jsr/detail?id=317)
entities.

### Background

The obligatory data stored with each media file is added
step-by-step during the import process:

* After the initial file upload, only data that can be extracted from
  the original file is available: File name, size, ...
* The thumbnail task processes the file and adds the paths to the generated
  thumbnail images.
* The proxy video task processes the file and adds the paths to the
  generated proxy videos.
* The user finishes the import, providing obligatory metadata on the
  dashboard, like the video title.

All obligatory fields should be **not null** once the import is finished.

### NOT NULL on the Database

On the database, `NOT NULL` constraints can be applied in a fully
normalized data model:

![/images/normalized-data-model.png](/images/normalized-data-model.png)

However, these are _four_ tables just for some basic data, and once
the import is completed, all four tables are joined all the time whenever a
media file is loaded from the data base.

### @NotNull on JPA Entities

With JPA's [@Embeddable](http://docs.oracle.com/javaee/6/api/javax/persistence/Embeddable.html)
annotation, it is easy to de-normalize the data model and to merge all data into
a single table:

![/images/denormalized-data-model.png](/images/denormalized-data-model.png)

However, in that case we cannot use database constraints to make sure
that the metadata is **not null** once the import is finished.

On the Java layer, [JSR 303 Bean Validation](http://jcp.org/en/jsr/detail?id=303)
can be used to anotate the mandatory fields with
[@NotNull](http://docs.oracle.com/javaee/6/api/javax/validation/constraints/NotNull.html):

{% highlight java %}
// ...
@Embeddable
public class MediaFileThumbnailData {
    @NotNull private String smallImg;
    @NotNull private String mediumImg;
    @NotNull private String largeImg;
    // ...
}
{% endhighlight %}

_But how can we make sure that the [@Embeddable](http://docs.oracle.com/javaee/6/api/javax/persistence/Embeddable.html)
objects are validated when the MediaFile is persisted?_

In order to enforce _cascaded validation_ of
[@Embeddable](http://docs.oracle.com/javaee/6/api/javax/persistence/Embeddable.html)
classes, the [@Valid](http://docs.oracle.com/javaee/6/api/javax/validation/Valid.html)
annotation must be used:

{% highlight java %}
@Entity
public class MediaFile {
    // ...
    @Valid private MediaFileProxyVideoData proxyVideoData;
    @Valid private MediaFileThumbnailData thumbnailData;
    // ...
}
{% endhighlight %}

That way, the **not null** constraints are enforced on the Java
entity layer whenever a MediaFile is persisted or updated.

### Result

Using Bean validation, an
[@Embeddable](http://docs.oracle.com/javaee/6/api/javax/persistence/Embeddable.html)
class can provide the same guarantees as real
[@Entity](http://docs.oracle.com/javaee/6/api/javax/persistence/Entity.html).
