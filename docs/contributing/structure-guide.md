---
title: Structure Guide
description: A comprehensive guide on JDA's codebase and how it is structured.
---

# How does JDA stay consistent in its code style and structure?

On this page we will try to concentrate all structure and code style guides that we ourself
use in JDA to ensure consistency and readability.

## Indentation and Brackets

In JDA we use a specific brackets placement that is common for C# development.
We put each `{` curly bracket on its own respective line:

!!! note inline end
    This rule goes for every curly bracket (open `{`/close `}`) and every scope usage such as try/catch/finally, loops, methods, lambda expressions, class scopes...

```java
public void someMethod()
{
    this.works.well();
    for (Each element : from()) // (1)
    {
        System.out.println(element);
    }
}
```

1.  Always put a space in-between the scope usage and the opening bracket (i.e. `if (true)` and not `if(true)`)

### Indentation

We only use indentation of **4 spaces** consistently throughout JDA.
<br>If a Pull Request does not use this indentation we will not accept it.

## Class Structure

In this section we guide you through a logically ordered and structured class under JDA's point of view.

### Access Modifiers

Access Modifiers are the keywords such as `public`, `protected` and `private`. They restrict other members from
accessing these fields, methods or classes from locations throughout the library.

When trying to order your fields, methods or nested classes we recommend using this logical order:

1. Public Members
2. Protected Members
3. Private Members
4. Package Private Members (no access modifier)

In addition it is recommended to always put `static` fields and methods (not [nested classes](#nested-classes)) first in your class.
<br>Fields marked with the `final` keyword should come first and should be separated from other fields.
<br>For better structure it is suggested to group fields by their declared types.

### Methods

Methods are always defined after fields and the constructor of your class.
<br>There are 3 types of methods listed in logical order of appearance:

1. Overriding or Implementing which make use of the `@Override` tag
2. Internal (impl) setters which can be found in the impl classes of JDA.
3. Object Overrides such as `toString`, `equals` and `hashCode`

### Nested Classes

Nested classes no matter if `static` or member should always be placed at the very bottom of your class.
<br>These include enums and other class types. It is also recommended following the access modifier (see above sections) order here again.

### Imports and Copyright

Every class in JDA has a **Copyright Header** (see example).
<br>Imports use wildcard `*` when they import 5+ classes from the same package.

## JavaDoc

We put JavaDoc on the following targets:

- Public Methods that directly confront the JDA user
- Class level if the specific class has API features that the user can interact with such as an entity or manager
- Package docs, all not excluded packages require a `package-info.java` class with proper JavaDoc

### Paragraphs

JDA has a specific JavaDoc structure that is unique to our repository.
<br>We use a style in which we encapsulate each important part of the doc in a "block" for itself.
<br>Each `<br>` tag is placed in-front of the new line: 

!!! note inline end
    `<br>` tags are not to be closed

```java
/**
 * This is my first line
 * <br>And this is my second line
 */
```

A new paragraph starts with `<p>` (not `<br><br>`!).
<br>The `<p>` tag is supposed to be placed either between the previous and following paragraph or like the `<br>` tag directly
in-front of the first line of that following paragraph:

!!! note inline end 
    Do not close a paragraph tag! It is unnecessary and redundant.

```java
/**
 * Either do this
 * 
 * <p>Or do this
 * <p>
 * You can decide.
 */
```

### Escaping

When you want to use characters that are not available in JavaDoc source most people tend to go with escaped characters such as `&tm;`.
<br>In JDA we usually stick -if possible- to `{@literal ™}` for readability sake. When using this JavaDoc tag you must remember
that it does not allow other nested tags since it will replace them with their literal characters instead!
<br>Another important tag you can use to achieve something that `<code>true</code>` does you can use `{@code true}`.
If you however want to also have nested links or other JavaDoc tags in your code snippet you can fallback to using the `<code>` tag:
```java
/**
 * JDA {@literal >} All others
 * <br>Because we can do `{@code channel.sendMessage("hey").queue()}` AND `{@code channel.sendMessage("hey").complete()}`!
 * 
 * <p>Or even do <code>channel.sendMessage("hey").{@link net.dv8tion.jda.core.requests.RestAction#submit submit()}</code>!!
 */
```

!!! note
    If you are using a tag like this, you **have** to close it!

### Linking

If the class is not already imported, use the fully qualified name when you are linking to something through JavaDoc!
<br>Bad: `{@link RestAction}`
<br>Good: `{@link net.dv8tion.jda.core.requests.RestAction RestAction}`

!!! note
    We also highly recommend setting an alias name as you can see in the 2nd example snippet.

When you link to an external resource (such as the official API docs) you can use the `<a>` tag to create
a hyperlink.
<br>It is recommended to use the `target="_blank"` tag.
<br>Example: `<a href="https://ci.dv8tion.net/job/JDA" target="_blank">Download JDA</a>`

**Hint**: Sometimes it helps to link other methods in the description and then also including an `@see #otherMethod` at the very bottom.

### Case: RestAction return type

When your method makes use of [RestAction](https://github.com/DV8FromTheWorld/JDA/tree/master/src/main/java/net/dv8tion/jda/core/requests/RestAction.java)
you have to document possible `ErrorResponses` that can follow from this request:

```java
/**
 * <p>The following {@link net.dv8tion.jda.core.requests.ErrorResponse ErrorResponses} are possible:
 * <ul>
 *     <li>{@link net.dv8tion.jda.core.requests.ErrorResponse#MISSING_ACCESS MISSING_ACCESS}
 *     <br>The request was attempted after the account lost access to the
 *         {@link net.dv8tion.jda.core.entities.Guild Guild} or {@link net.dv8tion.jda.client.entities.Group Group}
 *         typically due to being kicked or removed, or after {@link net.dv8tion.jda.core.Permission#MESSAGE_READ Permission.MESSAGE_READ}
 *         was revoked in the {@link net.dv8tion.jda.core.entities.TextChannel TextChannel}</li>
 *
 *     <li>{@link net.dv8tion.jda.core.requests.ErrorResponse#MISSING_PERMISSIONS MISSING_PERMISSIONS}
 *     <br>The send request was attempted after the account lost {@link net.dv8tion.jda.core.Permission#MESSAGE_WRITE Permission.MESSAGE_WRITE} in
 *         the {@link net.dv8tion.jda.core.entities.TextChannel TextChannel}.</li>
 * </ul>
 */
```

!!! note
    Always make these your last description paragraph
    <br>Taken from: [MessageChannel](https://github.com/DV8FromTheWorld/JDA/blob/master/src/main/java/net/dv8tion/jda/core/entities/MessageChannel.java#L169-L179)

The `@return` for simple RestActions is as follows:
```java
/**
 * @return {@link net.dv8tion.jda.core.requests.RestAction RestAction} - Type: String
 *         <br>This is an -optional- description of what this RestAction will provide in case the type isn't enough
 */
```

### Example Template

```java
    /**
     * This description should inform the user about the basic function of the method (or class)
     * that is being documented.
     * <br>A line break should be placed at the beginning of the following line.
     *
     * <p>This description is optional and should contain additional / notable information about
     * this method (or class)
     *
     * <p>All additional description paragraphs should start with the paragraph tag
     * at the beginning of the new paragraph and should be separated from the previous
     * paragraph by (at least) one line.
     *
     * <p>The last paragraph should point out what the possible {@link net.dv8tion.jda.core.requests.ErrorResponse ErrorResponses}
     * are. These can occur in RestAction failures.
     * <br>For that the following format should be used:
     * <ul>
     *     <li>{@link net.dv8tion.jda.core.requests.ErrorResponse#UNKNOWN_MESSAGE UNKNOWN_MESSAGE}
     *     <br>The Message did not exist (possibly deleted)</li>
     *
     *     <li>{@link net.dv8tion.jda.core.requests.ErrorResponse#INVALID_PIN INVALID_PIN}
     *     <br>The message specified can not be pinned (possibly a system message)</li>
     *
     *     <li>{@link net.dv8tion.jda.core.requests.ErrorResponse#MISSING_PERMISSIONS MISSING_PERMISSIONS}
     *     <br>This can be caused if we do not hold one of the following Permissions:
     *         <ul>
     *             <li>{@link net.dv8tion.jda.core.Permission#MESSAGE_WRITE MESSAGE_WRITE}
     *             <br>We are unable to send a message to this channel</li>
     *
     *             <li>{@link net.dv8tion.jda.core.Permission#MESSAGE_READ MESSAGE_READ}
     *             <br>We are unable to read messages in this channel</li>
     *         </ul></li>
     * </ul>
     *
     * @param  var0 (1)
     *         The Description should be at the same level as the parameter name
     * @param  var1
     *         Multiple parameters are to be documented in one "block"
     *
     * @throws javax.security.auth.login.LoginException
     *         The same goes for descriptions of throwables
     * @throws net.dv8tion.jda.core.exceptions.RateLimitedException
     *         Multiple throwables are to be documented in one "block"
     *
     * @return {@link net.dv8tion.jda.core.requests.RestAction RestAction} - Type: {@link net.dv8tion.jda.core.entities.Role Role}
     *         <br>The response type of the RestAction can be described further here.
     *
     * @see    Void
     * @see    net.dv8tion.jda.core.JDA
     *
     * @since  3.0
     *
     * @serialData
     *         If a tag is not specified here it should be at the bottom of the documentation.
     *         <br>If the tag name is too long to follow the proper indentation formatting
     *         it should start the block in the next line with the correct indentation.
     */
```

1.  We align each block with whitespace as you can see how `@param` is separated with 2 space characters from the actual parameter name