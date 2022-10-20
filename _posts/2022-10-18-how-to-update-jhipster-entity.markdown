---
layout: "post"
title:  "How to modify existing entity generated with jhipster?"
date:   "2022-10-18 15:39:00 -0500"
categories: jhipster, entity, docker
---

[How to modify existing entity generated with jhipster?](https://stackoverflow.com/questions/28216307/how-to-modify-existing-entity-generated-with-jhipster){:target="_blank"}

[JHipster](https://www.jhipster.tech/){:target="_blank"} uses Liquibase to manage database updates. The liquibase [diff](https://docs.liquibase.com/commands/diff/diff.html){:target="_blank"} command can be used detect a drift between a model schema and a database's actual schema.

Typically, to update a jhipster entity, you do the following:
- Make sure your database is running, then start the application to generate your tables.
- Stop the application but leave the database is running.
- Make changes to your jdl file, then re-import it to re-generate your entities. Don't overwrite any changelog (.xml files or .csv fake-data files) as they are immutable and their checksums are persisted.
- Run the command `./mvnw compile liquibase:diff` to compile your and generate a new changelog (difference between your entities and database). A new changelog will be generate at `src/main/resources/config/liquibase/changelog`
. Review it and add it to `src/main/resources/config/liquibase/master.xml`
- It is applied the next time your run the application


Done!