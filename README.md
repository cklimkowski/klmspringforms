# klmspringforms
<<<<<<< HEAD
KLM A simple form using Spring


If you want to embed images, this is how you do it:

![KLM package diagram](https://github.com/cklimkowski/klmspringforms/blob/master/wiki/klmpackages.png)

a simple controller
a few JSP views

=======
A simple form using Spring

The class tree and diagram looks like this: 
![KLM package diagram](https://github.com/cklimkowski/klmspringforms/blob/master/wiki/klmpackages.png)

>>>>>>> 3d2c0a135485f26210641b4dc2646f2887a9b2bb
The **UserManagementController** at the beginning consists of two private properties **userDatabase**, **userIdSequence** and a synchronized private method **getNextUserId()** which yields another user ID by incrementing **userIdSequence**. There is also a request handler method **displayUsers(map)** that retrieves a map of users from **userDatabase** property, saves it into a **model** map employing a **"userList"** string as id and returns the name of the proper view **"user/list"** as a string.

```java
@Controller
public class UserManagementController
{
    private final Map<Long, User> userDatabase = new Hashtable<>();
    private volatile long userIdSequence = 1L;

    @RequestMapping(value = "user/list", method = RequestMethod.GET)
    public String displayUsers(Map<String, Object> model)
    {
        model.put("userList", this.userDatabase.values());
        return "user/list";
    }
...
    private synchronized long getNextUserId()
    {
        return this.userIdSequence++;
    }
}

```
The **User** as a simple *POJO* has three private fields **userId**, **username**, **name** and the relevant mutators and accessors.

The *WEB-INF/jsp/view/user/list.jsp* does two things. First it enables adding new user using **<c:url>** tag to fill a value of *href* property inside a HTML link. Second it provides viewing the existing users (if exist any), utilizing **<c:forEach>** tag to iterate through a map of users **"${userList}"**.

```jsp
<%--@elvariable id="userList" type="java.util.Collection<klm.pstryk.site.User>"--%>
<!DOCTYPE html>
<html>
    <head>
        <title>User List</title>
    </head>
    <body>
        <h2>Users</h2>
        [<a href="<c:url value="/user/add" />">new user</a>]<br />
        <br />
        <c:forEach items="${userList}" var="user">
            ${user.name} (${user.username})
            [<a href="<c:url value="/user/edit/${user.userId}"/>">edit</a>]<br/>
        </c:forEach>
    </body>
</html>
```

Following we have another request handler method **createUser(map)** that constructs a new **UserForm** object, saves it into a **model** map as a **"userForm"**, returns the  **user/add** view name.
```jsp
    @RequestMapping(value = "user/add", method = RequestMethod.GET)
    public String createUser(Map<String, Object> model)
    {
        model.put("userForm", new UserForm());
        return "user/add";
    }
```
Next we have a similar request handler method **editUser(map model, long userId)**. It captures the **User** being edited from **userDatabase** map using **userId**, constructs and populates a new **UserForm** object, exposes to the model and returns the appropriate view name - **user/edit**.
```jsp
    @RequestMapping(value = "user/edit/{userId}", method = RequestMethod.GET)
    public String editUser(Map<String, Object> model,
                           @PathVariable("userId") long userId)
    {
        User user = this.userDatabase.get(userId);
        UserForm form = new UserForm();
        form.setUsername(user.getUsername());
        form.setName(user.getName());
        model.put("userForm", form);
        return "user/edit";
    }

```
Instead of the **UserForm** object I could easily use **User** object but I would like to distinguish form objects and business objects.

The views */WEB_INF/jsp/view/user/add.jsp* and */WEB_INF/jsp/view/user/edit.jsp* set the **title** Expression Language variable and include the */WEB_INF/jsp/view/user/form.jspf* that is a template for the form.

```jsp
<c:set var="title" value="Add User" />
<%@ include file="form.jspf" %>
```
```jsp
<c:set var="title" value="Edit User" />
<%@ include file="form.jspf" %>
```
```jsp
<%--@elvariable id="userForm" type="klm.pstryk.site.UserForm"--%>
<!DOCTYPE html>
<html>
    <head>
        <title>${title}</title>
    </head>
    <body>
        <h1>${title}</h1>
        <form:form method="post" modelAttribute="userForm">
            <form:label path="username">Username</form:label><br />
            <form:input path="username" /><br />
            <br />
            <form:label path="name">Name:</form:label><br />
            <form:input path="name" /><br />
            <br />
            <input type="submit" value="Save" />
        </form:form>
    </body>
</html>
```

To pick up the form submission data I need to employ HTTP POST method this is RequestMethod.POST in my case. The Spring converter called **FormHttpMessageConverter** (path ..) renders automatically the **UserForm** object from request parameters and the request handler methods **createUser(UserForm)**, **editUser(UserForm, long)** make use of it.
```jsp
    @RequestMapping(value = "user/add", method = RequestMethod.POST)
    public View createUser(UserForm form)
    {
        User user = new User();
        user.setUserId(this.getNextUserId());
        user.setUsername(form.getUsername());
        user.setName(form.getName());
        this.userDatabase.put(user.getUserId(), user);
        return new RedirectView("/user/list", true, false);
    }
```
```jsp
    @RequestMapping(value = "user/edit/{userId}", method = RequestMethod.POST)
    public View editUser(UserForm form, @PathVariable("userId") long userId)
    {
        User user = this.userDatabase.get(userId);
        user.setUsername(form.getUsername());
        user.setName(form.getName());
        return new RedirectView("/user/list", true, false);
    }
```

The Spring automatically bounds the **UserForm** object on the model to the form fields in the view. When editing an existing user, the values from the form **<form:form method="post" modelAttribute="userForm">** from the model attribute **"userForm"** are automtically copied into the relevant form fields.




