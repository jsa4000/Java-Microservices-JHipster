# Develop Java Microservices With JHipster

* [R.A.D.](https://en.wikipedia.org/wiki/Rapid_application_development)
* [Yeoman Generator](https://yeoman.io/generators/)
* [JHipster Generator](https://www.jhipster.tech/)

## Installation

In order to use JHipster, following requirements are needed:

* **Java 11** (Openjdk, sdkman, etc...)
* **Node.js** from the Nodejs website (LTS 64-bit version)
* **JHipster** globally installation. `npm install -g generator-jhipster`
* **Yeoman**, If you want to use a module or a blueprint. `npm install -g yo`

Optional installations:

* **Java build tool** depending on preferences or requirements use `Maven` or `Gradle`
* **Git** from git-scm.com. Use a tool like `SourceTree` if you are starting with Git.
* **Docker** to be able to use container images to run a local environment

## Monolith Sample

### Create JHipster Project

* Create a `blog` directory and create a JHipster app in it

  ```bash
  # Create an empty folder
  mkdir monolith-jhipster

  # Execute following command
  cd monolith-jhipster
  jhipster
  ```

  ```bash
  # Use following settings or default values for unknown fields
  applicationType: monolith
  name: blog
  package: org.jhipster.blog
  authenticationType: JWT
  prodDatabaseType: PostgreSQL
  languages: en,es
  testFrameworks: Cypress
  ```

* Check the `.yo-rc.json` file jhipster settings and values and `README.md` file.

* Start the application using `./mvnw`

* Browse through `Administrator` features (`admin/admin)` in the [dashboard](http://localhost:8080)
* Confirm everything works by running `Cypress`

  > Server must be running at http://localhost:8080

  ```bash
  npm run e2e
  ```

* Check JHipster generated code best features:

  * Patterns, AOP, Error handlers, Logging, etc..
  * Controllers, Repositories, Services, Entities
  * Migration, Faker data, Liquisbase
  * Build tool files (pom), Docker files, internalization, etc..
  * Configuration files
  * Boilerplate code generated and libraries used in the project.

### Generate Entities

* Import Blog JDL from [JHipster JDL Studio](https://start.jhipster.tech/jdl-studio/)

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

* Run the following command using the previous file imported

  > Use `a` option to overwrite all the files.

  `jhipster jdl blog.jdl`

* Check source code generated and compare with previous version (red color))

* Check the `.jhipster` folder to track the entities.

* Restart application and show *pre-loaded* data using `./mvnw`

* Turn off `faker` in `application-dev.yml` and `rm -rf target/h2db`

  ```yaml
  liquibase:
  # Add 'faker' if you want the sample data to be loaded automatically
  contexts: dev
  ```

* Create a couple of blogs and entries for `admin` and `user`

* Show how `admin` and `user` share data

* Commit changes into local git repository to compare with previous version. `Pre-hooks` are enabled by default.

  ```bash
  git add .
  git commit -m "Updated Blog Entities"
  ```

### Create custom Entity

For each entity you want to create:

* a database table;
* a *Liquibase* changeset;
* a JPA entity class;
* a Spring Data `JpaRepository` interface;
* a Spring MVC `RestController` class;
* an Angular list component, edit component, service; and
* several HTML pages for each component.

Create an entity using the following steps

* Create an entity using the command line

  ```bash
  # Ruj the following command
  jhipster entity book
  ```

  Answer the next questions concerning the fields of this entity, the book has:

  * “title”, of type “String”
  * “description”, of type “String”
  * “publicationDate”, of type “LocalDate”
  * “price”, of type “BigDecimal”

* Check the with the new entities created

* Check the current `entity` (domain) and `liquibase` migration files.

* Restart application and show *pre-loaded* data using `./mvnw`

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

### Code quality

Sonar is used to analyse code quality. You can start a local Sonar server (accessible on http://localhost:9001) with:

```bash
docker-compose -f src/main/docker/sonar.yml up
```

You can run a Sonar analysis with using the [sonar-scanner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner) or by using the maven plugin.

Then, run a Sonar analysis:

```bash
./mvnw -Pprod clean verify sonar:sonar
```

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

* Error: `Require statement not part of import statement  @typescript-eslint/no-var-requires`
  
  Try to remove that line since it is an experimental feature on jhipster 7 that allows compute code coverage using `Cypress`. However it seems not working using `Sonar` code quality rules.

  You can disable this check in `.eslintrc.js` file

  ```js
  module.exports = {
    ...
    rules: {
      ...
      "@typescript-eslint/no-var-requires": 0,
    }
  }
  ```

