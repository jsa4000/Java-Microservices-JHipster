# Develop Java Microservices With JHipster

* [Yeoman Generator](https://yeoman.io/generators/)
* [JHipster Generator](https://www.jhipster.tech/)
* [R.A.D.](https://en.wikipedia.org/wiki/Rapid_application_development)

## Monolith Sample

### Create JHipster Project

* Create a `blog` directory and create a JHipster app in it

  ```bash
  applicationType: monolith
  name: blog
  package: org.jhipster.blog
  authenticationType: JWT
  prodDatabaseType: PostgreSQL
  languages: en,es
  testFrameworks: Cypress
  ```

* Start app using `./mvnw`, browse through admin features

* Confirm everything works by running `Cypress`

### Generate Entities

* Import Blog JDL from [start.jhipster.tech](https://start.jhipster.tech)

  ```csharp
  entity Blog {
    name String required minlength(3)
    handle String required minlength(2)
  }

  entity Post {
    title String required
    content TextBlob required
    date Instant required
  }

  entity Tag {
    name String required minlength(2)
  }

  relationship ManyToOne {
    Blog{user(login)} to User
    Post{blog(name)} to Blog
  }

  relationship ManyToMany {
    Post{tag(name)} to Tag{post}
  }

  paginate Post, Tag with infinite-scroll
  ```

  `jhipster jdl blog.jdl`

* Restart application and show pre-loaded data

* Turn off `faker` in `application-dev.yml` and `rm -rf target/h2db`

* Create a couple of blogs and entries for `admin` and `user`

* Show how `admin` and `user` share data

### Add Business Logic

* Edit `BlogResource.java` and change `getAllBlogs()` method

  return blogRepository.findByUserIsCurrentUser()

* Show how blog list screen limits data to current user

* Edit `PostResource.java` and change `getAllPosts()`

  ```java
  page = postRepository.findByBlogUserLoginOrderByDateDesc(SecurityUtils.getCurrentUserLogin().orElse(null), pageable);
  ```

* Using your IDE, create this method in `PostRepository`

* Recompile both classes and verify entries are limited to current user

### Make UI Enhancements

* Allow HTML in entries with `[innerHTML]="post.content"`

* Improve entry layout to look like a blog

### Generate CI-CD

* Run following command and select the ci-cd tool and options.

  `jhipster ci-cd`

### Deploy to the Cloud

* Build for production

  `./mvnw -Pprod verify`

* Fix test failures by setting a `User` on a blog by default

  ```java
  @Autowired
  private UserRepository userRepository;

  public Blog createEntity(EntityManager em) {
      Blog blog = new Blog()
              .name(DEFAULT_NAME)
              .handle(DEFAULT_HANDLE)
              .user(userRepository.findOneByLogin("user").get());
      return blog;
  }
  ```

* Run build again

* Login to Heroku using `heroku login`

* Run `jhipster heroku`

* When process completes, run `heroku open`

* Finished

## Reactive Sample

### Creating the project

* Create the project from a file specified (`.jdl` or `.jh`)

  ```bash
  # Create an empty folder
  mkdir reactive-jhipster

  # Execute following command
  cd reactive-jhipster
  jhipster jdl reactive-ms

  # This takes the file directly from the jhipster examples repository:
  #    https://raw.githubusercontent.com/jhipster/jdl-samples/v7.1.0/reactive-ms.jdl
  ```

### Build Container images

* For each folder run `jibDockerBuild` to build the container images

  ```bash
  # Create gateway container image
  cd gateway
  ./gradlew bootJar -Pprod jibDockerBuild

  # Create blog container image
  cd blog
  ./gradlew bootJar -Pprod jibDockerBuild

  # Create store container image
  cd store
  ./gradlew bootJar -Pprod jibDockerBuild
  ```

* Check the images created

  ```bash
  # Check the images have been build
  docker images
  ```

### Deploy and Run the Applications

* Deploy the applications using docker-compose locally.

  ```bash
  # Execute the following command
  cd docker-compose
  docker-compose up

  # Or New docker method
  docker compose --project-name reactive-jhipster up
  ```

## FAQ

* Add `keycloak` dns entry to host file (/etc/hosts)

    `127.0.0.1 keycloak`

* Error building the container images or using webpack

    ```bash
    [webpack-cli] Failed to load '/Users/jsantosa/Projects/Java/jhipster/temp/reactive-jhipster/gateway/webpack/webpack.prod.js' config
    [webpack-cli] Error: Cannot find module 'workbox-build/build/options/schema/webpack-generate-sw'
    ```

    To solve replace following dependency in projects.

    `"workbox-webpack-plugin": "6.1.5"`

    to

    `"workbox-webpack-plugin": "6.2.4"`
