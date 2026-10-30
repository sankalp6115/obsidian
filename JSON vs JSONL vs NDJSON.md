00:00:00
 If you've worked with APIs, configuration files, or data pipelines, you've probably used JSON. It's simple, readable, and supported by almost every programming language. But when your data grows into millions of records, or you need to process data as it arrives, regular JSON can become less convenient. There's another format called JSON L that solves a very specific problem. It changes how records are organized, making it easier to process large data sets one record at a time. The interesting part is that the

00:00:34
 difference comes down to something as simple as a new line character. Let's see how JSON L works, where it fits into real applications, and when you should choose it over regular JSON. First, let's understand the difference between JSON and JSON L. JSON, or JavaScript Object Notation, represents structured data using objects and arrays. Suppose you're storing information about three customers. With regular JSON, you might place all three records inside a single array. The entire data set

00:01:06
 becomes one JSON document with an opening bracket, multiple records, and a closing bracket. JSON L, which stands for JSON Lines, organizes the same data differently. Instead of wrapping every record inside one large array, it stores each record as an independent JSON object on its own line. The first line might contain Alice's details, the second Bob's, and the third Carol's. Each line is a complete JSON record that can be parsed independently. The actual data hasn't changed. What's

00:01:39
 different is how the records are organized. That small change has important consequences for how applications read, process, and store data. Now, why does putting each record on a separate line matter? Let's say you're processing a file containing millions of customer records. With a traditional JSON array, a program using a buffering parser may need to read and parse the entire document before it can start processing the records. For sufficiently large data sets, this can consume substantial memory. JSON L

00:02:12
 makes incremental processing straightforward. Your application reads one line, parses that record, processes it, and moves to the next line. It doesn't need to keep the entire data set in memory just to work through the records. For example, imagine a 50 megabyte file containing customer transactions. A program that loads the entire JSON document into memory may need substantially more than the file size once the parsed objects and other overhead are included. With JSON L, a line-by-line processor can work with

00:02:45
 just the current record and whatever additional state its processing requires. The memory requirement therefore depends much more on the largest individual record and the application's processing logic than on the size of the entire file. There's one important distinction, though. Streaming parsers can also process a traditional JSON document incrementally, including nested arrays and objects. JSON L simply makes independent line-by-line processing easier to implement. But JSON L isn't just about saving

00:03:16
 memory. It also benefits from existing ecosystem support. You could create your own format by writing JSON objects on separate lines and splitting the file whenever you encounter a new line. Technically, that can work. But adopting a recognized format means you don't have to maintain a custom convention and explain it to every developer who encounters your data. JSON L already has support across a range of data processing tools. For example, Python's Pandas library can read JSON L using its read_json function

00:03:49
 with the lines option enabled. Apache Spark and Hadoop can process line-oriented JSON data, while command-line tools and data ingestion systems can work with individual records without requiring one enormous JSON array. This support reduces the amount of custom parsing code you need to maintain. You still need to handle malformed records and other errors appropriately, but you can build on established tooling rather than inventing your own format. Let's look at where JSON L becomes particularly useful.

00:04:20
 One of the clearest examples is application logging. Applications continuously generate events, warnings, and errors. Those records accumulate over time, and you often want to inspect or process them before the application stops running. JSON L works naturally for this use case because each log entry is independent. Your application can append a new record to the end of the file without rewriting the entire data set. Another example is machine learning. Training data sets can contain millions of examples, and loading every example

00:04:53
 into memory may be impractical. With JSON L, a training pipeline can read records incrementally and process them in batches. This is useful when working with data sets containing prompts, responses, labels, or other structured training examples. Data engineering pipelines can benefit as well. Suppose you're ingesting records from an external system, transforming them, and loading them into a destination. A JSON L file lets you append new records and process subsets without rebuilding one large JSON

00:05:25
 structure. The same idea applies to event streams and distributed data processing. Independent records are easier to process incrementally, and line boundaries can simplify splitting files into chunks. However, the format alone doesn't make a system distributed or guarantee parallel execution. Those capabilities depend on the tools and processing architecture you're using. So, should you replace every JSON file with JSON L? Definitely not. Regular JSON remains a great choice when your application needs a single

00:05:59
 structured document. For example, REST API is commonly exchange JSON objects and arrays. Browser applications can parse JSON directly using built-in JavaScript functionality. Configuration files are another natural fit. A configuration document is usually small enough to load entirely and nested objects make it easy to represent related settings. Document databases also benefit from rich nested structures when the application needs to represent relationships within an individual document. If your data set is small and

00:06:32
 comfortably fits in memory, JSON may be the simpler choice. JSON L becomes more attractive when you have large data sets, append-only records, or data that needs to be processed incrementally. The decision comes down to how you use the data. If you need one complete document, regular JSON is often appropriate. If you need to process independent records one at a time, JSON L is worth considering. Now, what about NDJSON? Is it different from JSON L? You might come across another name, NDJSON, which stands for newline

00:07:07
 delimited JSON. It describes essentially the same approach. Each line contains an independent JSON value, commonly one JSON object per line. The two formats have closely related conventions and their names are often used interchangeably in practice. You'll frequently see .jsonl and .ndjson as file extensions for these kinds of files. If you're working across different tools, check the format's specific requirements rather than assuming every implementation handles every edge case identically.

00:07:40
 For most everyday data processing scenarios, the central idea remains the same. Each line can be read and parsed independently. Here's what to remember. JSON and JSONL both represent structured data, but they organize multiple records differently. JSON works naturally for complete documents, API responses, and configuration files. JSONL is especially useful for large data sets, logs, and pipelines that process records incrementally. When you're designing a data pipeline, ask yourself one question. Do I need the

00:08:16
 entire document at once, or would independent records be easier to process? That decision can influence memory usage, streaming behavior, and how much custom parsing code you need to maintain. Sometimes, choosing the right data format is a small design decision that makes the entire pipeline easier to build and operate.

