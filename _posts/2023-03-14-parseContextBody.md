---
layout: single
title: "[webserv] Parser class - parseContextBody method"
---

# parseContextBody method from Parser class

"it to parse context body(http, server, location)."

|            |                  `type`                   |        `name`         |                `description`                 |
| :--------: | :---------------------------------------: | :-------------------: | :------------------------------------------: |
|   return   | std::pair<bool, std::vector\<Directive> > |                       |                                              |
| parameters |                std::string                |      config_path      |   config path for printing error messages    |
|            |            enum DirectiveKind             |         kind          | current block directive kind context parsing |
| variables  |          std::vector\<Directive>          |      directives       |   simple directive vector for return value   |
|            |        std::pair<bool, Directive>         | simple_directive_pair |                                              |



### Entire code snippet for parseContextBody method

```c++
std::pair<bool, std::vector<Directive> > Parser::parseContextBody(std::string config_path, enum DirectiveKind kind)
{
	std::vector<Directive>		directives;
	std::pair<bool, Directive> 	simple_directive_pair = expectSimpleDirective(config_path, kind);

	while (simple_directive_pair.first == true) // parse every simple directive until it encounters any block directive
	{
		directives.push_back(simple_directive_pair.second);
		simple_directive_pair = expectSimpleDirective(config_path, kind);
	}
	/*
	 there are two cases when it returns false
	 1. it is a block directive(http, server, location)
	 2. it is an invalid directive or unknown directive
	*/
	if (simple_directive_pair.first == false) // check if it is a block directive or an error
	{
		if (kind == HTTP)
		{
			if (simple_directive_pair.second.directive == SERVER)
				return (std::make_pair(true, directives)); // when it's a valid server directive
			if (simple_directive_pair.second.directive == HTTP ||
						simple_directive_pair.second.directive == LOCATION)
			{
				std::cout << "Error: \"" << sDirectiveKindStrings[simple_directive_pair.second.directive];
				std::cout << "\" directive is not allowed here in ";
				std::cout << config_path << ":" << current_token_->line_num << std::endl;
				return (std::make_pair(false, directives));
			}
		}
		else if (kind == SERVER)
		{
			if (simple_directive_pair.second.directive == LOCATION)
				return (std::make_pair(true, directives)); // when it's a valid location directive
			if ((simple_directive_pair.second.directive == SERVER) ||
					(simple_directive_pair.second.directive == HTTP))
			{
				std::cout << "Error: There can't be any other block than location in server block, current directive is `";
				std::cout << sDirectiveKindStrings[simple_directive_pair.second.directive] << "` in line ";
				std::cout << current_token_->line_num << ".\n";
				return (std::make_pair(false, directives));
			}
		}
		if (kind == LOCATION && ((simple_directive_pair.second.directive == SERVER) ||
					(simple_directive_pair.second.directive == HTTP) ||
					(simple_directive_pair.second.directive == LOCATION)))
		{
			std::cout << "Error: There can't be any other block in location block.\n";
			return (std::make_pair(false, directives));
		}
		// until now there isn't any block directive and invalid directive
		// check if it encloses with }
		if (expectToken(OPERATOR, "}").first == true)
		{
			--current_token_;
			return (std::make_pair(true, directives));
		}
	}
	return (std::make_pair(false, directives));
}
```



#### while loop from the code snippet

```c++
while (simple_directive_pair.first == true) // parse every simple directive until it encounters any block directive
{
	directives.push_back(simple_directive_pair.second);
	simple_directive_pair = expectSimpleDirective(config_path, kind);
}
```

[line 1 ~ 5]  parse every simple directive until it encounters any block directive(http, server, location)



#### if statement from the code snippet

```c++
if (simple_directive_pair.first == false) // check if it is a block directive or an error
{
	if (kind == HTTP)
	{
		if (simple_directive_pair.second.directive == SERVER)
			return (std::make_pair(true, directives)); // when it's a valid server directive
		if (simple_directive_pair.second.directive == HTTP ||
					simple_directive_pair.second.directive == LOCATION)
		{
			std::cout << "Error: \"" << sDirectiveKindStrings[simple_directive_pair.second.directive];
			std::cout << "\" directive is not allowed here in ";
			std::cout << config_path << ":" << current_token_->line_num << std::endl;
			return (std::make_pair(false, directives));
		}
	}
	else if (kind == SERVER)
	{
		if (simple_directive_pair.second.directive == LOCATION)
			return (std::make_pair(true, directives)); // when it's a valid location directive
		if ((simple_directive_pair.second.directive == SERVER) ||
				(simple_directive_pair.second.directive == HTTP))
		{
			std::cout << "Error: There can't be any other block than location in server block, current directive is `";
			std::cout << sDirectiveKindStrings[simple_directive_pair.second.directive] << "` in line ";
			std::cout << current_token_->line_num << ".\n";
			return (std::make_pair(false, directives));
		}
	}
	if (kind == LOCATION && ((simple_directive_pair.second.directive == SERVER) ||
				(simple_directive_pair.second.directive == HTTP) ||
				(simple_directive_pair.second.directive == LOCATION)))
	{
		std::cout << "Error: There can't be any other block in location block.\n";
		return (std::make_pair(false, directives));
	}
	// until now there isn't any block directive and invalid directive
	// check if it encloses with }
	if (expectToken(OPERATOR, "}").first == true)
	{
		--current_token_;
		return (std::make_pair(true, directives));
	}
}
return (std::make_pair(false, directives));
```

[line 3 ~ 15] it checks when the value of `kind` is HTTP and the block directive is server, it returns true with parsed simple directives. otherwise, it returns false printing an error message.

[line 16 ~ 28] when the value of `kind` is SERVER and the block directive is location, it returns true with parsed simple directives. otherwise, it returns false printing an error message.

[line 29 ~ 31] when the value of `kind` is LOCATION and there are any block directives, it returns false printing an error message.

[line 33 ~ 37]  when there are no block directive encountered or no invalid simple directives ahead, it verifies if it is enclosed with a closing curly brace.