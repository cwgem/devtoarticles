{%- # TOC start (generated with https://github.com/derlin/bitdowntoc) -%}

- [Naming Convention Documentation](#naming-convention-documentation)
- [Typing Documentation](#typing-documentation)
- [Comment Documentation](#comment-documentation)
- [Project README](#project-readme)
- [Internals Documentation](#internals-documentation)
- [Process Documentation](#process-documentation)
- [General Usage Documentation](#general-usage-documentation)
- [API Documentation](#api-documentation)
- [Advanced Documentation](#advanced-documentation)
- [External Documentation](#external-documentation)
- [Hosting Documentation](#hosting-documentation)
- [Conclusion](#conclusion)

{%- # TOC end -%}

Documentation is extremely valuable for both open source and internal company projects. With that in mind the question arises on what exactly needs to be documented. While code docs generated from inline formats (ex. python doc strings) may be what is considered to be documentation for developers, there's more to it than that. In this article I'll be breaking down different kinds of documentation and what gives them value to their target audience. Layout wise I'll start at the lowest level and move upwards.

## Naming Convention Documentation

First off is the naming of variables and functions which drive the code. A common mistake for newer developers is having code with a lot of inline comments. Much of these verbose comments could be addressed by simply having a naming convention which provides clarity on what the purpose of the function or variable is. As an example:

- `isAddressPOBox()`
- `Address.isPOBox()`
- `ec2_boto_client`

The first and second example shows how naming can change between context. Meanwhile `ec2_boto_client` could be useful if you're dealing with multiple boto clients connecting to the AWS API. Something to note here is that `isAddressPOBox()` could technically just be `isPOBox` but I find that the verbose version also gives context on the input. I would say that you want to get into the habit of doing this for even personal pet projects. It's also very valuable in team environments to make code easier to read (and potentially faster PRs).

## Typing Documentation

Typing is useful for giving hints on function input and output. While this is default for statically typed languages, some dynamic languages such as python may support type hinting features for this particular purpose:

```python
def average_numbers(numbers: list[int]) -> float:
```

In this example I know I need to provide a list of integers as input and will obtain a float as output. You will want to be careful on how you organize custom types. A lack of organization can lead to a lot of back and forth on source code reading. Another nice benefit of typing is that it can be used by many IDEs to enhance linting features by seeing if inputs and outputs match the requested types. This is useful for cases where your project is a dependency library as well as working on team environments.

## Comment Documentation

Documentation through code comments can be useful in cases where code may rely on complex algorithms or when something is done in an unusual manner. I find most of the code I comment on tends to be related to cryptographic hashing:

```python
).sign(self._key_object, hashes.SHA256())
# SHA256 was chosen here instead of SHA512 as a compromise between decrypt performance and
# hash security
```

 In this example I'm explaining the reasoning why I'm using SHA256 to sign a cert when the more powerful SHA512 hash exists. `TODO` comments are a specialized form of comment documentation which allows developers to keep track of code improvements quickly, which can be transitioned to tickets/issues/tasks later on. I will warn on being careful on your ratio of comments to code. At a certain point the actual code becomes difficult to follow and could be a sign that your naming convention/typing might have issues.

 ## Application Interface Documentation

 This style of documentation is often what developers consider to be "code documentation". Users may reference it for libraries to figure out what functions need to be called. Developers may use it to understand the code base. Such documentation is often found after a function/method signature to document inputs, outputs, and a basic summary of what the code does:

 ```python
 def average_numbers(numbers: list[int]) -> float:
    """Average a list of numbers

    :param numbers: The list of numbers to average
    :type numbers: list[int]

    :return: The average of the numbers
    :rtype: float
    """
    return np.average(numbers)
 ```

 Important to note is that the format of application interface documentation will vary depending on the programming language and documentation solution. If you find that files become too verbose due to application interface documentation it might be a sign of functions trying to do too much or a need for separation refactoring. 

## Project README

I would consider this the starting point of figuring out basic information for a project. This includes items such as:

- Why the project was created
- Status of the project
- Project requirements
- How to install the project (or a link to an install guide for more complex projects)
- Basic usage / getting started
- Security policy
- Links to further documentation
- Contact information (filing bug reports, etc.)

A well laid out README can often be a great way to get project adoption as it brings an assumption that the rest of the project is well documented and users will find information they need easily.

## Internals Documentation

This somewhat ties in with application interface documentation. It's general used to supplement such documentation and primarily geared towards developers, power users, and potential contributors to a project. A few examples of internals documentation:

- [Gitea Actions Design Docs](https://docs.gitea.com/usage/actions/design)
- [Kubernetes Helm Architecture](https://v2.helm.sh/docs/architecture/)
- [Python Developer's Guide](https://devguide.python.org/)

In general this type of documentation tends to be useful at larger scale products with wide user adoption. Understanding internals of a project can also help in finding potential performance optimizations or conditions that might cause bugs to occur. 

## Process Documentation

This is used to describe various processes a project might have in dealing with the codebase. Some examples of this include:

- How to contribute to a project
- Setting up a development environment
- Project code of conduct (in particular enforcement of it)
- Bug reporting
- Security reporting guidelines (different from basic bug reporting as it tends to be done in a more private manner to avoid early exploits in the wild)
- Release process

It's essentially how users might interact with a project versus how to use the actual project itself. This category of documentation is primarily oriented towards projects with a high contribution rate.

## General Usage Documentation

General usage documentation is geared towards end users of the project. This may replace parts of the README documentation if certain usage details require a substantial explanation. General usage documentation often touches on the following components:

- Downloading the project (source code/package/installer)
- Installing the project
- Configuring the project
- Validating the project (generally in the form of a quick start guide)

Application interface documentation may be included here as well if the project in question is primarily used as a dependency. Smaller projects may combine download, installation, and configuration steps. Larger projects may need them broken out to handle environment variations such as operating systems, package managers, service management, database engines, etc.

## API Documentation

Often used for cases where a project exposes a REST or other type of API service. [Open API](https://spec.openapis.org/oas/latest.html) is a popular method of documenting such API services. It can also be used along side tools such as [Swagger Codegen](https://swagger.io/tools/swagger-codegen/) to produce boilerplate code for API interaction / testing purposes. There may also be support files for popular API testing tools such as [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/). This makes it easier at a glance to see what data is coming back from a call so the user knows how to handle parsing the data.

## Advanced Documentation

General usage documentation works well for covering the standard user needs for the project. Advanced documentation enhances this by showing more advanced setups which are useful but not what you would expect from an out of the box setup. Such documentation could touch on manual configurations that are normally set to developer recommended settings to cover a majority of use cases. Finally there are cases where subject matter expertise is required such as BGB network integration with Kubernetes networking providers. Such documentation is generally recommended for more active projects where the developer is more aware of project use cases.

## External Documentation

This may also be considered "community documentation". Someone writes a blog post on a project, maybe a key note video is posted, essentially anything that isn't directly hosted on core project infrastructure. It may also point to community resources such such as forums, slack channels, and social media sites. One word of caution is that I recommend avoiding slack exclusive documentation for a project as there is a barrier of entry in accessing the information (not to mention separation of concerns). 

## Hosting Documentation

Once you have all the documentation worked out a place to host it will be necessary. Some documentation generation may have ties in with specific hosting sites. [Read The Docs'](https://about.readthedocs.com/?ref=readthedocs.com) support for Sphinx and other documentation tools is one example. [GitHub pages](https://pages.github.com/) can be useful for GitHub hosted projects as it integrates well with GitHub Actions CI/CD deployments.

If you want to self host and don't mind paying a bit a combination of Route53 (unless you host DNS elsewhere), S3, and CloudFront on AWS can provide a pretty reliable and cost effective site hosting. It's also a great solution if you're working on an internal company project with architecture hosted on AWS. It's also nice in compliance environments since you can implement strict access controls when necessary which can integrate with company SSO and other IAM components.

Wikis are another solution if your contributors are more comfortable with them. It also helps prevent the issue of "documentation updates caused a big CI run" that can catch some projects off guard. For GitHub users there is a fairly minimal [GitHub Wiki feature](https://docs.github.com/en/communities/documenting-your-project-with-wikis/about-wikis) for projects that support it. This also works on the enterprise versions if you're doing an internally hosted project.

## Conclusion

As someone who has been a proponent of documentation for much of my career, I hope this provides some insight on the depth of documentation you can have for a project. I will say that some types of documentation tend to show their true value at certain project growth stages. Trying to do advanced usage documentation is not ideal when your project is starting out and you don't have enough usage information. With that said I recommend looking over each documentation type and evaluating if it would help with project adoption / contributions.