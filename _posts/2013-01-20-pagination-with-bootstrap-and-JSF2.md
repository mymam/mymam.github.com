---
layout: post
category : programming
tags : [bootstrap, JSF2]
image: /images/paginator-screenshot.png
author: Fabian Stäber
---
{% include JB/setup %}

Overview
--------

This post shows how to implement Twitter Bootstrap's
[paginator](http://twitter.github.com/bootstrap/components.html#pagination)
as a composite component in
[JSF 2](http://docs.oracle.com/javaee/6/tutorial/doc/bnaph.html)-based applications.

<img src="{{page.image}}"/>

The composite component was developed as part of MyMAM.
Feel free to copy the files listed at the end of this post
and use them in your own project.

Including the Paginator in an XHTML-Facelet
-------------------------------------------

The paginator is used as follows:

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
            "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:mymam="http://java.sun.com/jsf/composite/composites">

    <!--
        ...
        Your paginatable content here, using #{yourPaginatableBean}
        as the backing bean.
    -->
    <mymam:paginator
            paginatable="#{yourPaginatableBean}"
            size="9"
            prevLabel="prev"
            nextLabel="next"/>
    </html>

An example can be found in MyMAM's [index.xhtml](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/webapp/index.bootstrap.xhtml).

Implementing a Paginatable Backing Bean
---------------------------------------

The `paginatable` parameter in the XHTML-Facelet is the backing bean for the component to be paginated.
That can be any bean implementing the 
[Paginatable](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/java/net/mymam/controller/Paginatable.java)
interface:

{% highlight java %}
public interface Paginatable {

    // total number of pages
    public int getNumberOfPages();

    // current page number
    public int getCurrentPage();

    // set the current page
    public void selectPage(int page);
}
{% endhighlight %}

Some notes on the implementation:

- Pages are numbered starting with `1`, i.e. the first page is `1`, not `0`.
- The `selectPage()` method can be implemented using lazy loading.
- The scope of the `Paginatable` bean must be
  [@ViewScope](http://docs.oracle.com/javaee/6/api/javax/faces/bean/ViewScoped.html)
  or larger, because that bean must store the current page number between pagination requests.

List of Files
-------------

If you want to use the paginator in your project, copy the following files form the MyMAM repository:

- [paginator.xhtml](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/webapp/resources/composites/paginator.xhtml):
  the composite component.
- [PaginatorBean.java](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/java/net/mymam/controller/PaginatorBean.java):
  the backing bean for the paginator component.
- [PaginatorLinkBean.java](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/java/net/mymam/controller/PaginatorLinkBean.java):
  the backing bean for a single page link within the paginator.
- [Paginatable.java](https://github.com/mymam/mymam/blob/master/mymam-server/src/main/java/net/mymam/controller/Paginatable.java):
  the Paginatable interface described above.

<hr/>
<span style="font-size: 0.8em; color: #696969;">Author: {{page.author}}</span>
