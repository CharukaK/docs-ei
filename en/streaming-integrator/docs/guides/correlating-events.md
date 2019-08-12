#Correlating Data

The streaming integrator can correlate data in order to detect patterns and trends in streaming data. Correlating can be done via patterns as well as sequences.

The difference between patterns and sequence is that sequences require all the matching events to arrive consecutively to
 match the sequence condition, whereas patterns identify events that match the pattern condition irrespective of the order in which they arrive.

## Correlating events to find a pattern

This section explains how you can use Siddhi patterns to detect trends and patterns. There are two types of Siddhi patterns as follows:

- Counting Patterns: These count the number of intances that match the given pattern condition.
- Logical Patterns: These identify logical relationships between events.

### Count and match multiple events for a given pattern condition

To understand how to count and match multiple events that match a specific condition, consider the example where a store 
wants to check the frequency with which a specific product needs to be repaired within two months after it is purchased. 
If a specific product is brought back for repairs within two months more than five times, the manager of purchases needs 
to be notified via a mail. To do this, create a Siddhi application as follows.

1. Start creating a new Siddhi application. You can name it `DefectDetectionApp` For instructions, see [Creating a Siddhi Application](../develop/creating-a-Siddhi-Application.md).
    `@App:name("DefectDetectionApp")`
    
2. Define the input streams into which the events compared are received.
    1. To capture information about purchases, define a stream as follows.
        `define stream PurchasesStream (productName string, custID string);`
    2. To capture information about repairs, define a stream as follows.
        `define stream RepairsStream (productName string, custID string);`
        
3. To notify the purchase manager that the threshold is reached define an output sink with an email sink attached as follows:
    ```
    @sink(type='email', address='storemanager@abc.com', username='storemanager', password='secret_password', subject='Defective Product Alert', to='purchasemanager@headoffice.com', @map(type = 'text', @payload("Hello,The product {{productName}} is identified as defective.\n\nThis message was generated automatically.")))
    define stream DefectiveProductsStream (productName string);
    ```
    
4. To count occurrences where a product is brought back for repairs withing two months following its purchase, and identify 
products where the treshold for such occurrences is reached, create a query as follows.

    1. To specify the input streams from which the input events to be analyzed for pattern detection are taken, add a `from` clause as follows.
        `from every (e1=PurchasesStream) -> e2=RepairsStream[e1.productName==e2.productName and e1.custID==e2.custID]<5:> within 2 months`
        
        !!!info
            Note the following about the above `from` clause:
            - The input is derived from two streams. Therefore, first, both streams considered are specified and a unique reference is assigned to each stream. The `PurchasesStream` is referred to as `e1` and the `RepairsStream` is referred to as `e2`.
            - The matching condition to be met is that both streams should have an event where the values for both `productName` and `custID` attributes are the same.
            - The event in the `PurchasesStream` stream need to arrive before the matching event in the `RepairsStream` stream.
            - The matching event in the `RepairsStream` stream should arrive within two months after the arrival of the event in the `PurchasesStream` stream.
            - `<5:>` indicates that an output is generated only when the matching condition is met five times.
            - A time window of `2 months` is added to consider only a period of two months in a sliding manner when counting the number of times the matching condition for the pattern is met. For more information about time windows, see the [Siddhi Query Guide - Calculate and store clock time-based aggregate values](https://ei.docs.wso2.com/en/next/streaming-integrator/guides/summarizing-data/#calculate-and-store-clock-time-based-aggregate-values).
       
    2. To specify how the value for each attribute in the `DefectiveProductsStream` output stream is defined, add the `select` clause as follows.
        `select e1.productName`
        
       The output should only consist of the product identified to be possibly defective. Therefore, only the `productName` attribute is selected.
       
    3. To specify that the output has to directed to the `DefectiveProductsStream `, add the `insert into` clause as follows.
        `insert into DefectiveProductsStream`
        
The completed Siddhi application is as follows.
```sql
@App:name("DefectDetectionApp")


define stream PurchasesStream (productName string, custID string);

define stream RepairsStream (productName string, custID string);

@sink(type='email', address='storemanager@abc.com', username='storemanager', password='secret_password', subject='Defective Product Alert', to='purchasemanager@headoffice.com', @map(type = 'text', @payload("Hello,The product {{productName}} is identified as defective.\n\nThis message was generated automatically.")))
define stream DefectiveProductsStream (productName string);

from every (e1=PurchasesStream) -> e2=RepairsStream[e1.productName==e2.productName and e1.custID==e2.custID]<5:> within 2 months
select e1.productName
insert into DefectiveProductsStream
```

!!!INFO
    For more information, see [Siddhi Query Guide - Counting Patterns](https://siddhi.io/en/v4.x/docs/query-guide/#counting-pattern).
    
### Combine several patterns logically and match events

To understand how to combine several patterns logically and match events, consider an example of a factory foreman who 
needs to observe the factory output, identify any production decreases and check whether those decreases have reached 
maximum threshold which requires him to take action. To do this, you can create a Siddhi application as follows:

1. Start creating a new Siddhi application. You can name it `ProductionDecreaseDetectionApp` For instructions, see [Creating a Siddhi Application](../develop/creating-a-Siddhi-Application.md).
    `@App:name("ProductionDecreaseDetectionApp")`

2. Define an input stream as follows to capture the factory output.
    `define stream ProductionStream(productName string, factoryBranch string, productionAmount long);`

3. Now define an output stream as follows to present the observed production trend after applying the logical pattern.
    ```
    @sink(type='log', prefix='Decrease in production detected:')
    define stream ProductionDecreaseAlertStream (productName string, originalAmount long, laterAmount long, factoryBranch string);
    ```
    The output directed to this stream is published via a sink of the `log` type. For more information about publishing data via sinks, see the [Publishing Data guide](publishing-data.md).

4. To apply the pattern so that the production trend can be observed, add the `from` clause as follows.
    ```
    from every (e1=ProductionStream) -> e2=ProductionStream[e1.productName == e2.productName and e1.productionAmount - e2.productionAmount > 10]
         within 10 min
    ```
    !!!info
        Observe the following about the `from`clause:
        - Here, two events from the same stream are compared to identify whether the production has decreased. The unique reference for the first event is `e1`, and the unique reference for the second event is `e2`.
        - `e2` arrives after `e1`, but it is not necessarily the event that arrives immediately after `e1`.
        - The condition that should be met for `e1` and `e2` to be compared is `e1.productName == e2.productName and e1.productionAmount - e2.productionAmount > 10`. 
        This means, both the events should report the production of the same product, and there should be a decrease in 
        production that is greater than 10 between the `e1` and `e2` events.
        - A `10 min` time window is included to indicate that an output event is generated only if the decrease in production by 10 or more units takes place every ten minutes in a sliding manner. For more information about time windows, see the [Siddhi Query Guide - Calculate and store clock time-based aggregate values](https://ei.docs.wso2.com/en/next/streaming-integrator/guides/summarizing-data/#calculate-and-store-clock-time-based-aggregate-values).

5. To present the required output by deriving values for the attributes of the `ProductionDecreaseAlertStream` output stream you created, add the `select` clause as follows.
    `select e1.productName, e1.productionAmount as originalAmount, e2.productionAmount as laterAmount, e1.factoryBranch`
    
    Here, the production amount of the first event is presented as `originalAmount`, and the amount of the second event is presented as `laterAmount`.

6. To insert the output into the `ProductionDecreaseAlertStream` output stream, add the `insert into` clause as follows.
    `insert into ProductionDecreaseAlertStream;`
    
The completed Siddhi application is as follows.
```
@App:name("ProductionDecreaseDetectionApp")


define stream ProductionStream(productName string, factoryBranch string, productionAmount long);

@sink(type='log', prefix='Decrease in production detected:')
define stream ProductionDecreaseAlertStream (productName string, originalAmount long, laterAmount long, factoryBranch string);

from every (e1=ProductionStream) -> e2=ProductionStream[e1.productName == e2.productName and e1.productionAmount - e2.productionAmount > 10] within 10 min
select e1.productName, e1.productionAmount as originalAmount, e2.productionAmount as laterAmount, e1.factoryBranch
insert into ProductionDecreaseAlertStream;
```


## Find non-occurance of events

This section explains how to detect non-occurring events.

##Correlating events to find a trend(sequence)

This section explains how you can use Siddhi sequences to detect trends in events that arrive in a specific order. There are two types of Siddhi sequences as follows:

- Counting Sequences: These count the number of intances that match the given sequence condition.
- Logical Sequences: These identify logical relationships between events.

### Count and match multiple events for a given trend
### Combine several trends logically and match events

##Correlating two streams of data and unify

For a detailed explanation, see [Enriching Data - Enrich data by connecting with another stream of data](enriching-data/#enrich-data-by-connecting-with-another-stream-of-data)

## Correlate a stream and a static datasource to enrich
For a detailed explanation, see[Enriching Data - Enrich data by connecting with a data store](enriching-data/#enrich-data-by-connecting-with-a-data-store)