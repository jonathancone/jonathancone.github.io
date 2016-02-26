---
layout: post
title: 'HibernateException: Found two representations of same collection'
date: '2012-08-24T14:25:00.001-05:00'
tags: 
modified_time: '2012-08-24T14:25:14.827-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-1702647052905407366
blogger_orig_url: http://www.machineversus.me/2012/08/hibernateexception-found-two.html
---

Last week one of our application workflows stopped working out of the blue.  We began to see the following Hibernate exception: 
{% highlight java %}
org.hibernate.HibernateException: Found two representations of same collection: x.y.z.SomeClass.someAssociation
 at org.hibernate.engine.Collections.processReachableCollection(Collections.java:175)
 at org.hibernate.event.def.FlushVisitor.processCollection(FlushVisitor.java:60)
 at org.hibernate.event.def.AbstractVisitor.processValue(AbstractVisitor.java:122)
 at org.hibernate.event.def.AbstractVisitor.processValue(AbstractVisitor.java:83)
 at org.hibernate.event.def.AbstractVisitor.processEntityPropertyValues(AbstractVisitor.java:77)
 at org.hibernate.event.def.DefaultFlushEntityEventListener.onFlushEntity(DefaultFlushEntityEventListener.java:165)
 at org.hibernate.event.def.AbstractFlushingEventListener.flushEntities(AbstractFlushingEventListener.java:219)
 at org.hibernate.event.def.AbstractFlushingEventListener.flushEverythingToExecutions(AbstractFlushingEventListener.java:99)
 at org.hibernate.event.def.DefaultFlushEventListener.onFlush(DefaultFlushEventListener.java:50)
 at org.hibernate.impl.SessionImpl.flush(SessionImpl.java:1216)
    ... <more app specific classes below> ...
{% endhighlight %}

Other people have indeed seen this exception, but the solutions they used to fix it did not make sense with how we were using Hibernate.  I was baffled, I poured over our version control history to see what changeset could have possibly introduced this. Nothing. There were no changes to any of the application code that was involved in this workflow.  I was stuck, so I sat on it for a while hoping I would figure it out after sleeping on it.

A few days went by until one of our developers came up to me with a problem.  He had been working on shoring up our test coverage and told me another part of the application was having problems now and he would need my help to troubleshoot.  I took a look at a stack trace that he had encountered: 

{% highlight java %}
Caused by: org.hibernate.InstantiationException: No default constructor for entity: x.y.z.SomeOtherClass
 at org.hibernate.tuple.PojoInstantiator.instantiate(PojoInstantiator.java:107)
 at org.hibernate.tuple.component.AbstractComponentTuplizer.instantiate(AbstractComponentTuplizer.java:102)
 at org.hibernate.type.ComponentType.instantiate(ComponentType.java:515)
 at org.hibernate.type.ComponentType.instantiate(ComponentType.java:521)
 at org.hibernate.type.ComponentType.resolve(ComponentType.java:613)
 at org.hibernate.engine.TwoPhaseLoad.initializeEntity(TwoPhaseLoad.java:139)
 at org.hibernate.loader.Loader.initializeEntitiesAndCollections(Loader.java:982)
 at org.hibernate.loader.Loader.doQuery(Loader.java:857)
 at org.hibernate.loader.Loader.doQueryAndInitializeNonLazyCollections(Loader.java:274)
 at org.hibernate.loader.Loader.doList(Loader.java:2533)
 at org.hibernate.loader.Loader.listIgnoreQueryCache(Loader.java:2276)
 at org.hibernate.loader.Loader.list(Loader.java:2271)
 at org.hibernate.loader.hql.QueryLoader.list(QueryLoader.java:452)
 at org.hibernate.hql.ast.QueryTranslatorImpl.list(QueryTranslatorImpl.java:363)
 at org.hibernate.engine.query.HQLQueryPlan.performList(HQLQueryPlan.java:196)
 at org.hibernate.impl.SessionImpl.list(SessionImpl.java:1268)
 at org.hibernate.impl.QueryImpl.list(QueryImpl.java:102)
{% endhighlight %}

This immediately struck me as problematic because I knew that SomeOtherClass was an @Embedded Hibernate entity and required a public no-arg constructor.  I opened up the Java class and saw this: 

{% highlight java %}
@Embeddable
public class SomeOtherClass implements Comparable<SomeOtherClass> {

  private Integer listTypeId;

  @Column(name = "listContactId")
  private Integer contactId;

  public SomeOtherClass(int contactId, int listTypeId) {
    this.contactId = contactId;
    this.listTypeId = listTypeId;
  }

  public Integer getListTypeId() {
    return listTypeId;
  }
  ...
{% endhighlight %}

Someone had added an overloaded constructor to take two arguments.  This of course shadowed the default no-arg constructor which was no longer part of the API.  After some digging and discussion with my teammate, we saw that the overloaded constructor had been added to make unit testing the class easier.  The fix was as simple as re-adding the public no-arg constructor.  After I did this, I thought I might as well test the first workflow that I was stumped on and see if this changed the behavior of that.  You know what? It did.  The first workflow began working again.