layout: "post"
title: "Comparator && Comparable in Java"
date: "2018-11-06 16:33"
categories:
- [JAVA,DATA_STRACTURE]
tags:
- [TECHNOLOGY]
---
　　Comparator和Comparable是Java中将对象按照数据成员进行排序的两个接口，最近在读**《数据结构与算法分析　Java语言描述》_-Mark Allen Weiss_** ,看到书中的例子使用到了这两个接口，所以就想学习一下Java中这两个接口的使用方式　(｀・ω・´)
<!-- more -->
　　在Java中Comparable接口定义为:  
{% codeblock  lang:java %}
    package java.lang;
    import java.util.*;

    public interface Comparable<T> {    
      public int compareTo(T o);
    }
{% endcodeblock %}
　　接口中仅定义了一个***compareTo()***方法，自定义的对象则可以通过实现这个方法并结合Collections.sort()进行排序。比如自定义一个Order类:
{% codeblock  lang:java %}
  public class Order implements Comparable<Order>{

      private int orderNumber;
      private String orderDes;
      private LocalDate orderTime;

      public Order(int orderNumber, String orderDes, LocalDate orderTime) {
          this.orderNumber = orderNumber;
          this.orderDes = orderDes;
          this.orderTime = orderTime;
      }

      @Override
      public int compareTo(Order o) {
          return this.getOrderNumber() - o.getOrderNumber();
      }

      @Override
      public String toString() {
          return  this.orderNumber + "  " + this.orderDes + "  " + this.orderTime;
      }
      // ... 省略getter setter
{% endcodeblock %}
　　该Order类中定义了三个字段orderNumber、orderDes、orderTime，依次表示订单号、订单描述、订单时间。***compareTo()***方法简单的返回订单号的值的相减结果，第一次单元测试用例如下:
{% codeblock lang:java %}
    @Test
    void testComparableSimple() {
        ArrayList<Order> lists = new ArrayList<>();
        lists.add(new Order(123,"order 123",LocalDate.of(2018,11,6)));
        lists.add(new Order(221,"order 221",LocalDate.of(2012,9,1)));
        lists.add(new Order(71,"order 71",LocalDate.of(2015,2,7)));
        lists.add(new Order(691,"order 691",LocalDate.of(2011,6,21)));
        lists.add(new Order(419,"order 419",LocalDate.of(2009,3,31)));
        for (int i = 0; i < lists.size() - 1 ; i++) {
            System.out.println("第" + i +"个元素与第" + (i+1) + "元素进行比较:" +
                    lists.get(i).compareTo(lists.get(i+1)));
        }
    }
  /* 输出结果:
      第0个元素与第1元素进行比较:-98
      第1个元素与第2元素进行比较:150
      第2个元素与第3元素进行比较:-620
      第3个元素与第4元素进行比较:272
  */
{% endcodeblock %}
　　从上面的输出结果可以看出，Order类实现的Comparable接口的作用就是用于与其它Order对象使用***compareTo()***进行比较并返回一个int值。目前我们还看不出这个方法的作用，但是我们再看下一个单元测试就会明白其的作用了:
{% codeblock lang:java %}
  @Test
  void testComparableSecond() {
      ArrayList<Order> lists = new ArrayList<>();
      lists.add(new Order(123,"order 123",LocalDate.of(2018,11,6)));
      lists.add(new Order(221,"order 221",LocalDate.of(2012,9,1)));
      lists.add(new Order(71,"order 71",LocalDate.of(2015,2,7)));
      lists.add(new Order(691,"order 691",LocalDate.of(2011,6,21)));
      lists.add(new Order(419,"order 419",LocalDate.of(2009,3,31)));

      System.out.println("未进行排序前的顺序:");
      for(Order order: lists){
          System.out.println(order);
      }
      // 调用Collections.sort()进行排序
      Collections.sort(lists);
      System.out.println("进行排序之后的顺序:");
      for(Order order: lists){
          System.out.println(order);
      }
  }
  /* 输出结果:
      未进行排序前的顺序:
      123  order 123  2018-11-06
      221  order 221  2012-09-01
      71  order 71  2015-02-07
      691  order 691  2011-06-21
      419  order 419  2009-03-31
      进行排序之后的顺序:
      71  order 71  2015-02-07
      123  order 123  2018-11-06
      221  order 221  2012-09-01
      419  order 419  2009-03-31
      691  order 691  2011-06-21
  */
{% endcodeblock %}
　　从输出结果可以看出lists里的数据根据orderNumber进行了升序排序，而这是因为Order实现了Comparable接口的***compareTo()***方法,我们在该方法中返回相减的结果。当结果为正数，零，或者负数分别表示调用***compareTo()***的对象大于、等于、小于传递到方法中的参数对象，默认升序排序。如果需要降序排序则返回相反数即可。
　　至于Comparator接口，如果需要使用其来进行排序则需要编写一个类去实现它的***compare()***方法，这里我们编写一个先根据订单号再根据订单日期进行排序的实现类***SortByOrderNumberAndTime.class:***
{% codeblock lang:java %}
  public class SortByOrderNumberAndTime implements Comparator<Order> {
        /**
         * 根据订单号进行排序，订单号相同则根据时间进行排序
         * @param o1
         * @param o2
         * @return
         */
        @Override
        public int compare(Order o1, Order o2) {
            int numberOrder = o1.getOrderNumber() - o2.getOrderNumber();
            int timeOrder = o1.getOrderTime().compareTo(o2.getOrderTime());
            if(numberOrder == 0 && timeOrder != 0)
                return timeOrder;
            else
                return numberOrder;
        }
  }
{% endcodeblock %}
　　对应的单元测试:
{% codeblock lang:java %}
  @Test
  void testComparator() {
      ArrayList<Order> lists = new ArrayList<>();
      lists.add(new Order(123,"order 123",LocalDate.of(2018,11,6)));
      lists.add(new Order(221,"order 221",LocalDate.of(2012,9,1)));
      lists.add(new Order(221,"order 221",LocalDate.of(2017,2,7)));
      lists.add(new Order(221,"order 221",LocalDate.of(2015,6,21)));
      lists.add(new Order(419,"order 419",LocalDate.of(2009,3,31)));
      System.out.println("未进行排序前的顺序:");
      for(Order order: lists){
          System.out.println(order);
      }
      Collections.sort(lists,new SortByOrderNumberAndTime());
      System.out.println("排序之后的顺序");
      for(Order order: lists){
          System.out.println(order);
      }
  }
/*输出结果:
  未进行排序前的顺序:
  123  order 123  2018-11-06
  221  order 221  2012-09-01
  221  order 221  2017-02-07
  221  order 221  2015-06-21
  419  order 419  2009-03-31
  排序之后的顺序
  123  order 123  2018-11-06
  221  order 221  2012-09-01
  221  order 221  2015-06-21
  221  order 221  2017-02-07
  419  order 419  2009-03-31
*/
{% endcodeblock %}
　　我们可以看到lists首先根据订单号然后根据时间先后顺序进行了排序，这里也是依据SortByOrderNumberAndTime类中实现的***compare()***方法进行的排序，并且在compare()里使用到了LocalDate类里的compareTo()方法去比较时间先后顺序，除了LocalDate,Java中许多常用的类也默认提供了对Comparable接口的实现，比如String、Integer、Character、Float、Byte基本数据类型的包装类，具体可以查看官网的Docs。
　　不论是Comparator接口还是Comparable接口，都是实现排序的一种方式，当然也可以结合一起使用，具体的使用方式还是得取决于最终的需求。
