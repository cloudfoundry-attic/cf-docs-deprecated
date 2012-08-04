---
title: Play Java App TodoList on Cloud Foundry
description: Play Java Application with Persistent Backend Deployed on Cloud Foundry
tags:
    - play
    - java
    - postgresql
    - mysql
    - tutorial
---

This is a guide describing in more detail the TodoList app, use cases it covers and
various components.

## App use case
TodoList is used to create and manage tasks.

![todolist-usecase.png](/images/play/todolist-usecase.png)

These tasks are created by the users and the data
is stored in underlying relational store.
The default tutorial stores tasks in the embedded database which comes with the Play framework.

## Components of the App

The App has four main components:

+    Routes
+    Controller
+    Model
+    View

### Router
Routing information of the app is described in the `conf/route` file. It maps the HTTP path
to the corresponding actions in the Controller.

``` javascript
# Home page
GET     /                           controllers.Application.index()

# Map static resources from the /public folder to the /assets URL path
GET     /assets/*file               controllers.Assets.at(path="/public", file)

# Tasks
GET     /tasks                  controllers.Application.tasks()
POST    /tasks                  controllers.Application.newTask()
POST    /tasks/:id/delete       controllers.Application.deleteTask(id: Long)
```

### Controller
The app has a single controller `controllers.Application.java` which has actions mapped to HTTP
paths as described in the routing file below:

``` java
package controllers;

import play.*;
import play.mvc.*;
import play.data.*;
import models.Task;

import views.html.*;

public class Application extends Controller {
  static Form<Task> taskForm = form(Task.class);
  public static Result index() {
    return redirect(routes.Application.tasks());
  }

  public static Result tasks() {
    return ok(views.html.index.render(Task.all(), taskForm));
  }

  public static Result newTask() {
    Form<Task> filledForm = taskForm.bindFromRequest();
  if(filledForm.hasErrors()) {
    return badRequest(
      views.html.index.render(Task.all(), filledForm)
    );
  } else {
    Task.create(filledForm.get());
    return redirect(routes.Application.tasks());
  }
  }

  public static Result deleteTask(Long id) {
    Task.delete(id);
    return redirect(routes.Application.tasks());
  }
}
```

### Model
Model is made up of a Single Object `models.Task`. The app uses JPA to identify the Id and the
required fields.

``` java
package models;

import java.util.*;
import play.data.validation.Constraints.*;
import play.db.ebean.*;
import javax.persistence.*;

@Entity
public class Task extends Model{
  public static Finder<Long,Task> find = new Finder(
    Long.class, Task.class
  );

  @Id
  public Long id;
  @Required
  public String label;

  public static List<Task> all() {
    return find.all();
  }

  public static void create(Task task) {
    task.save();
  }

  public static void delete(Long id) {
    find.ref(id).delete();
  }
}
```
### View
View is a scala function in Play with a mix of HTML and scala statements. We are passing the
List of tasks and taskForm object into it. The view shows a list of existing tasks and the
 ability to create and delete tasks.
``` html
@(tasks: List[Task], taskForm: Form[String])

@import helper._

@main("Todo list") {

    <h1>@tasks.size task(s)</h1>

    <ul>
        @tasks.map { task =>
            <li>
                @task.label

                @form(routes.Application.deleteTask(task.id)) {
                    <input type="submit" value="Delete">
                }
            </li>
        }
    </ul>

    <h2>Add a new task</h2>

    @form(routes.Application.newTask) {

        @inputText(taskForm("label"))

        <input type="submit" value="Create">

    }
}
```
## SQL Evolutions
Play 2.0 generates the SQL evolution file based on the Object model designed.

### Default Evolution File
The code shown below is the default evolution file `1.sql` generated in `evolutions/default` folder.

``` sql
 # --- !Ups

create table task (
  id                        bigint not null,
  label                     varchar(255),
  constraint pk_task primary key (id))
;

create sequence task_seq;


# --- !Downs

SET REFERENTIAL_INTEGRITY FALSE;

drop table if exists task;

SET REFERENTIAL_INTEGRITY TRUE;

drop sequence if exists task_seq;

```