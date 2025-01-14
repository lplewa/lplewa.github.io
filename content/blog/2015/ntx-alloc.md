---
# Blog post title
title: 'An introduction to pmemobj (part 5) - atomic dynamic memory allocation'

# Blog post creation date
date: 2015-06-18T19:55:17-07:00

# Change to 'false' when publishing the blog post
draft: false

# Blog post description
description: ''

# Blog post hero image. Used to override the default hero background image.
# eg: image: "/images/my_blog_heroimg.png"
hero_image: ''

# Blog post thumbnail
# eg: image: "/images/my_blog_thumbnail.png"
image: ''

# Blog post author
author: 'pbalcer'

# Categories to which this blog post belongs
blogs: ['libpmemobj']

tags: []

# Redirects from old URL
aliases: ['/2015/06/18/ntx-alloc.html']

# Blog post type
type: 'post'
---

In the previous post I talked about using transactions for allocating new objects, which is fine and is the most similar approach to the standard POSIX way. But it does add an overhead of maintaining an undo log of changes. A more optimal memory management can be achieved using the non-transactional atomic API the pmemobj library provides.

### Fail-safe atomic allocations

This API is **not** similar to the APIs most programmers are used to when it comes to handling memory. First of all, the functions are either allocating to, or freeing from a pointer. The modification of the destination pointer is done in an atomic way so that it is always valid - it either points to a valid and initialized memory block or an `OID_NULL`. The functions/macros also force you to create the objects in a known state - either by zeroing them (`POBJ_ZNEW`, `POBJ_ZALLOC`), or providing a constructor for you to initialize the pointer (`POBJ_NEW`, `POBJ_ALLOC`).

The rectangle example using this API looks like this:

{{< highlight C "linenos=table" >}}
int rect_construct(PMEMobjpool *pop, void *ptr, void *arg) {
struct rectangle *rect = ptr;
rect->x = 5;
rect->y = 10;
pmemobj_persist(pop, rect, sizeof \*rect);

    return 0;

}
POBJ*NEW(pop, &D_RW(root)->rect, struct rectangle, rect_construct, NULL);
int p = perimeter_calc(D_RO(root)->rect);
/* busy work \_/
POBJ_FREE(&D_RW(root)->rect);
{{< /highlight >}}

Certainly looks different, whether it's more complicated that's for you to decide. But in terms of performance, if you allocate a non-trivial number of objects - there may a significant benefit of adopting this approach. It also allows for more fine-grained control of using `_persist` functions - they are required in the constructor - this is especially beneficial when operating on large objects, because you can flush only the part you really need.

The constructor function also allows the programmer to cancel an in-progress
allocation by returning a non-zero value. This might be useful in cases in which
the constructor does a non-trivial work that depends on a different finite
resource - for example, this allows you to perform a volatile allocation inside
the constructor and back off if that volatile allocation didn't succeed.

The destination pointer is optional, or can be a variable on stack - the only correct way of accessing objects allocated that way are internal collections.

### Internal collections

I didn't introduced this topic before because I didn't want to confuse anyone. All of the existing objects are stored in a collection which you can access by using `POBJ_FIRST` and `POBJ_NEXT` APIs. This is done so that you never lose a reference to an object, i.e. have a persistent memory leak. You can also use this as an unordered list for iterating through your objects, without linking it with the root object.

For example, to access the rectangle from the previous example, following expression can be used:

{{< highlight C "linenos=table" >}}
TOID(struct rectangle) rect = POBJ_FIRST(pop, struct rectangle);
{{< /highlight >}}

If you had more then one `struct rectangle` then:

{{< highlight C "linenos=table" >}}
rect = POBJ_NEXT(rect, struct rectangle);
{{< /highlight >}}

will allow you access them. But you don't have to iterate objects by hand, there are macros for that:

{{< highlight C "linenos=table" >}}
TOID(struct rectangle) iter;
POBJ_FOREACH_TYPE(pop, iter) {
int p = perimeter_calc(D_RO(iter));
printf("Perimeter of rectangle = %d", p);
}
{{< /highlight >}}

This will iterate over the collection of `struct rectangle` objects, if you want to iterate over everything there's `POBJ_FOREACH` macro for that. A `SAFE` variant of both macros is also available that allows you to deallocate all of the objects.

### PMEM Invaders

A good example of non-transactional API usage can be found [here](https://github.com/pmem/pmdk/tree/master/src/examples/libpmemobj/pminvaders). Check out the way `POBJ_FOREACH_TYPE` macros are used in the application to access the objects.

###### [This entry was edited on 2017-12-11 to reflect the name change from [NVML to PMDK](/blog/2017/12/announcing-the-persistent-memory-development-kit).]
