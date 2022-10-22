---
layout: "post"
title:  "How to modify existing entity generated with jhipster?"
date:   "2022-10-18 15:39:00 -0500"
categories: jhipster, entity, docker
---

[How to modify existing entity generated with jhipster?](https://stackoverflow.com/questions/28216307/how-to-modify-existing-entity-generated-with-jhipster){:target="_blank"}

[JHipster](https://www.jhipster.tech/){:target="_blank"} uses Liquibase to manage database updates. It uses the [liquibase-hibernate5](https://github.com/liquibase/liquibase-hibernate){:target="_blank"} plugin, a liquibase extension for hibernate integration.

The [liquibase:diff](https://docs.liquibase.com/commands/diff/diff.html){:target="_blank"} maven goal is used to detect changes between your entities and database structure.

Typically, to update a jhipster entity, you do the following:
- Make sure your database is running, then start the application to generate your tables.
- Stop the application but leave the database is running. The reason is that you first want to persist your enities state in the database so that you can compare it with any new changes to your entities.
- Make changes to your jdl file, then re-import it to re-generate your entities. Don't overwrite any changelog (.xml files or .csv fake-data files) as they are immutable and their checksums are persisted.
- Run the command `./mvnw clean compile liquibase:diff` to compile your code and generate a new changelog (difference between your entities and database structure). The changelog will be generate at `src/main/resources/config/liquibase/changelog`
. Review it and add it to `src/main/resources/config/liquibase/master.xml` so it is applied next time you run your application.
- Note that liquibase generates an empty `alterSequence` changeset when using PostgreSQL. [This is a known bug](https://github.com/liquibase/liquibase/issues/2223){:target="_blank"} . You can manually delete it from the changelog.

```xml
<changeSet author="eosimosu (generated)" id="1666299093643-1">
    <alterSequence sequenceName="sequence_generator"/>
</changeSet>
```

Done!