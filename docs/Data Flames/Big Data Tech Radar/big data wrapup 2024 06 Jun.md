# Big Data Wrap-up of June 2024

### Rockset <a href="#rockset" id="rockset"></a>

**OpenAI acquires Rockset**

[OpenAI acquires RocksetOpenAI](https://openai.com/index/openai-acquires-rockset/)

LinkedIn FollowFeed Architectur

[Feed InfrastructureLinkedIn](https://engineering.linkedin.com/teams/data/data-infrastructure/feed-infrastructure)

Facebook Multifeed Archtecture

[Serving Facebook Multifeed: Efficiency, performance gains through redesignEngineering at Meta](https://engineering.fb.com/2015/03/10/production-engineering/serving-facebook-multifeed-efficiency-performance-gains-through-redesign/)

### Apache DataFusion <a href="#apache-datafusion" id="apache-datafusion"></a>

[Apache DataFusion — Apache DataFusion documentation](https://datafusion.apache.org/)

[Apache Arrow DataFusion: A Fast, Embeddable, Modular Analytic Query Engine | Companion of the 2024 International Conference on Management of DataACM Conferences](https://dl.acm.org/doi/10.1145/3626246.3653368)

[2024-06-11 SIGMOD\_ Apache Arrow DataFusion\_ A Fast, Embeddable, Modular Analytic Query Engine.pdfpdf](<files/2024-06-11 SIGMOD_ Apache Arrow DataFusion_ A Fast, Embeddable, Modular Analytic Query Engine.pdf>)



### Apache Arrow Ballista <a href="#apache-arrow-ballista" id="apache-arrow-ballista"></a>

[Apache Arrow Ballista — Apache Arrow Ballista documentation](https://arrow.apache.org/ballista/)

#### Overview <a href="#overview" id="overview"></a>

Ballista is a distributed compute platform primarily implemented in Rust, and powered by Apache Arrow.

Ballista has a scheduler and an executor process that are standard Rust executables and can be executed directly, but Dockerfiles are provided to build images for use in containerized environments, such as Docker, Docker Compose, and Kubernetes. See the [deployment guide](https://arrow.apache.org/ballista/user-guide/introduction.html#deployment.md) for more information

SQL and DataFrame queries can be submitted from Python and Rust, and SQL queries can be submitted via the Arrow Flight SQL JDBC driver, supporting your favorite JDBC compliant tools such as [DataGrip](https://arrow.apache.org/ballista/user-guide/introduction.html#datagrip) or [tableau](https://arrow.apache.org/ballista/user-guide/introduction.html#tableau). For setup instructions, please see the [FlightSQL guide](https://arrow.apache.org/ballista/user-guide/flightsql.html).

The scheduler has a web user interface for monitoring query status as well as a REST API.

#### How does this compare to Apache Spark? <a href="#how-does-this-compare-to-apache-spark" id="how-does-this-compare-to-apache-spark"></a>

Although Ballista is largely inspired by Apache Spark, there are some key differences.

* The choice of Rust as the main execution language means that memory usage is deterministic and avoids the overhead of GC pauses.
* Ballista is designed from the ground up to use columnar data, enabling a number of efficiencies such as vectorized processing (SIMD and GPU) and efficient compression. Although Spark does have some columnar support, it is still largely row-based today.
* The combination of Rust and Arrow provides excellent memory efficiency and memory usage can be 5x - 10x lower than Apache Spark in some cases, which means that more processing can fit on a single node, reducing the overhead of distributed compute.
* The use of Apache Arrow as the memory model and network protocol means that data can be exchanged between executors in any programming language with minimal serialization overhead.

### Apache DataFusion Comet <a href="#apache-datafusion-comet" id="apache-datafusion-comet"></a>

[https://datafusion.apache.org/comet/](https://datafusion.apache.org/comet/)

Apache DataFusion Comet is an Apache Spark plugin that uses Apache DataFusion as a native runtime to achieve improvement in terms of query efficiency and query runtime.


