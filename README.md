# Spring Cloud Zull #

Roadmap

* Overview Spring Cloud Zipkin 
* Overview Spring Cloud Sleuth
* Zipkin and Sleuth Integration
* Build from scratch
* Live Demo

## Spring Cloud Zipkin ##

* Zipkin was originally developed at Twitter
* Efficient tools for distributed tracing in microservices ecosystem
    * latency measurement
    * how much time
* Helps in identifying that slow component among in the ecosystem, by filter or sort all traces based on the application

### Internally it has 4 modules ###

* Collector - Once any component send the trace data to Zipkin collector daemon, it is validated, store & indexed for lookups by the Zipkin collector
* Storage - This module store and index the lookup data in backend. (Cassandra, Elastic Search & MySQL are supported)
* Search - This module provides a simple JSON API
* Web UI - A very nice UI Interface for viewing traces

## Using Zipkin ##

Just add some depndency on `pom.xml`in the spring boot project

    ```
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-server</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-autoconfigure-ui</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```

## Spring Cloud Sleuth ##

* Distributed tracing solution for Spring Cloud
    * trace id
    * span id
    * add these information to the service calls in the header and MDC
* It can be used by ZIPKIN
* It automatically integrated to the common communication channels
    * request made with the RestTemplate
    * request that pass throught a Netflix Zull Proxy
    * HTTP headers received at Spring MVC controller
    * Request over messaging technologies like Apache Kafka or RabbitMQ etc
    
### Using Sleuth ###

Is very easy. Just need to add it's `pom.xml` in the spring boot project

    ```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
    ```


## Build dan Run ##

### Spring Boot Initializr ###

1. Browse ke [https://start.spring.io/] (https://start.spring.io/)

2. Lengkapi Project Metadata

    Group
    
    ```
    com.sheringsession.balicamp.zipkin    
    ```

    Artifact
    
    ```
    demo
    ```
   
    Dependencies
    
    ```
    Web, JPA, Cache, Lombok, MySQL (Optional tergantung DB yang digunakan)
    ```
   
3. Generate Project

4. Download dan import pada IDE masing-masing


### Build with Maven ###

1. Pastikan `pom.xml` terdapat dependency berikut
    
    ```
    <dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId> 
			<scope>runtime</scope> 
        </dependency>
        <dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
    </dependencies>
    ```

2. `applicatioin.properties` untuk konfigurasi database source, hibernate dll

    ```
    spring.datasource.url=jdbc:mysql://localhost/spring_cahce_demo
    spring.datasource.username=disesuaikan
    spring.datasource.password=disesuaikan
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true
    
    spring.datasource.max-wait=40000
    spring.datasource.test-on-borrow=true
    spring.datasource.validation-query=SELECT 1
    
    spring.jackson.serialization.indent-output=true
    ```


### Build with your IDE ###

1. Buat Enity misal Product

    ```
    @SuppressWarnings("serial")
    @Data @Entity @Table
    public class Product implements Serializable{
        @Id @GeneratedValue(generator="uuid") @GenericGenerator(name="uuid", strategy= "uuid2")
        private String id;
        private String code;
        private String name;
        private BigDecimal amount;
        private Boolean outOfStock;
    }
    ```

2. Jalankan aplikasi

    ```
    mvn clean spring-boot:run
    ```
    Cek tabel Product otomatis akan terbentuk dalam database

3. Buat DAO `ProductDao` 

    ```
    public interface ProductDao extends PagingAndSortingRepository<Product, String>{
        public Product findByName(String name);
        public Product findByCode(String code);
    }
    ```

4. Buat Service `ProductService`

    ```
    public class ProductService {
        private static final Logger LOG = LoggerFactory.getLogger(ProductService.class);
        @Autowired private ProductDao productDao;
    
        public Product findProductByName(String name) {
            LOG.info("### Call service product by name {}", name);
            return productDao.findByName(name);
        }
        
        public Product findProductByCode(String code) {
            LOG.info("### Call service product by code {}", code);
            return productDao.findByCode(code);
        }
        
        public Product updateProduct(String id, String name) {
            LOG.info("### Call service put product id {}, name {}", id, name);
            Product p = productDao.findOne(id);
            p.setName(name);
            productDao.save(p);
            return p;
        }
        
        public void deleteProduct(String name) {
            LOG.info("### Call service evict product name {}", name);
        }
    }
    ```

4. Buat Controller `ProductController`

    ```
    @RestController @RequestMapping("/spring-cache/product")
    public class ProductController {
        private static final Logger LOG = LoggerFactory.getLogger(ProductController.class);
        @Autowired private ProductService productService;

        @GetMapping("/getByName/{name}")
        public Product getDataProductByName(@PathVariable String name) {
            LOG.info("## product controller getByName called here!!");
            return productService.findProductByName(name);
        }

        @GetMapping("/getByCode/{code}")
        public Product getDataProductByCode(@PathVariable String code) {
            LOG.info("## product controller getByCode called here!!");
            return productService.findProductByCode(code);
        }

        @GetMapping("/updateProduct/{id}/{name}")
        public Product updateDataProduct(@PathVariable String id, @PathVariable String name) {
            LOG.info("## product controller put product called here!!");
            return productService.updateProduct(id, name); 
        }

        @GetMapping("/deleteProduct/name/{name}")
        public String deleteDataProduct(@PathVariable String name) {
            LOG.info("## product controller delete product called here!!");
            productService.deleteProduct(name);
            return "Delete Product Cache berhasil !!!";
        }
    }
    ```
    
5. Jalankan menggunakan maven dan browse ke [http://localhost:8080/getByName/foo]

## GO TO LAB ... ##

### PENTING!!! @EnableCaching ###

1. Tambahkan anotasi `@EnableCaching` pada root/main Class Application Spring Boot

    ```
    @SpringBootApplication
    @EnableCaching
    public class DemoApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```

### Penggunaan @Cacheable ###

1. Tambahkan `@Cacheable("products")` pada salah satu method class ProductService

    ```
    @Cacheable("products")
	public Product findProductByCode(String code) {
		LOG.info("### Call service product by name {}", name);
		return productDao.findByName(name);
	}
    ```
    
2. Tambahkan `@Cacheable(cacheNames="products", key = "#code")` pada salah satu method class ProductService

    ```
    @Cacheable(cacheNames = "products", key = "#code")
	public Product findProductByCode(String code) {
		LOG.info("### Call service product by code {}", code);
		return productDao.findByCode(code);
	}
    ```

3. Tambahkan `@Cacheable(cacheNames = "products", key = "#code", condition = "#code!='p001'")` pada salah satu method class ProductService

    ```
    @Cacheable(cacheNames = "products", key = "#code", condition = "#code!='p001'")
	public Product findProductByCode(String code) {
		LOG.info("### Call service product by code {}", code);
		return productDao.findByCode(code);
	}
    ```

4. Tambahkan `@Cacheable(cacheNames = "products", key = "#name", unless = "#result.code == 'p001'")` pada salah satu method class ProductService

    ```
    @Cacheable(cacheNames = "products", key = "#name", unless = "#result.code == 'p001'")
	public Product findProductByName(String name) {
		LOG.info("### Call service product by name {}", name);
		return productDao.findByName(name);
	}
    ```
    
### Penggunaan @CachePut ###

* Tambahkan anotasi pada salah satu service berikut


    ```
    @CachePut(cacheNames="products", key = "#name")
	public Product updateProduct(String id, String name) {
		LOG.info("### Call service put product id {}, name {}", id, name);
		Product p = productDao.findOne(id);
		p.setName(name);
		productDao.save(p);
		return p;
	}
    ```

### Penggunaan @CacheEvict ###

* Tambahkan anotasi pada salah satu service berikut


    ```
    @CacheEvict(cacheNames="products", key = "#name")
	public void deleteProduct(String name) {
		LOG.info("### Call service evict product name {}", name);
	}
    ```

* Untuk semua cache, penggunaan `@CacheEvict` dengan mengaktifkan parameter `allEntries=true` 


    ```
    @CacheEvict(cacheNames="products", allEntries = true)
	public void deleteProduct(String name) {
		LOG.info("### Call service evict product name {}", name);
	}
    ```

### Penggunaan @CacheConfig ###

* Before
    
    ```
    @Service
    public class ProductService {
        ...
        ...
    }
    ```

* After

    ```
    @Service
    @CacheConfig(cacheNames = "products")
    public class ProductService {
        ...
        ...
    }
    ```

* Hapus semua cacheNames yang didaaftarkan pada class method:

    ```
    cacheNames="products"
    ```
    
    
## HOW NEXT ? ##

### Cache Providers ###

* Referensi : [https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html] (https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)

    * Generic
    * JCache (JSR-107)
    * EhCache 2.x
    * Hazelcast
    * Infinispan
    * Couchbase
    * Redis
    * Caffeine
    * Guava (deprecated)
    * Simple
    * None
    
GOTO Example. . .

### Hazelcast Caching ###

Mencoba salah satu Provider Spring Cache : [https://memorynotfound.com/spring-boot-hazelcast-caching-example-configuration/] (https://memorynotfound.com/spring-boot-hazelcast-caching-example-configuration/) 

1. Menambahkan maven dependency pada `pom.xml`


    ```
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast-spring</artifactId>
        <version>3.8.6</version>
    </dependency>
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast</artifactId>
        <version>3.8.6</version>
    </dependency>
    ```
2. Membuat configurasi class Hazelcast

    ```
    @Configuration
    public class HazelcastConfig {
        @Bean
        public Config haselcastConfig() {
            return new Config()
                    .setInstanceName("hazelcast-instance")
                    .addMapConfig(
                            new MapConfig()
                                .setName("products")
                                .setMaxSizeConfig(
                                    new MaxSizeConfig(
                                            200, 
                                            MaxSizeConfig.MaxSizePolicy.FREE_HEAP_SIZE
                                            )
                            )
                            .setEvictionPolicy(EvictionPolicy.LRU)
                            .setTimeToLiveSeconds(20)
                    );

        }
    }
    ```

3. Init `CacheManager` pada Controller

    ```
    @Autowired private CacheManager cacheManager;
    ```

4. Contoh test penggunaan cache yang telah di provide dengan hazelcast

    ```
    @GetMapping("/getByName/{name}")
	public Product getDataProductByName(@PathVariable String name) {
		...
		LOG.info("PROVIDER YANG DIGUNAKAN : {}",cacheManager.getClass().getName());
		...
	}
    ```
    
5. Implement Serializable class entity
    
    ```
    public class Product implements Serializable{
        ...
    }
    ```
    
### FINISH ###