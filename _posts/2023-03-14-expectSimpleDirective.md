---
layout: single
title: "[webserv]:config Parser class - expectSimpleDirective method"


---

# expectSimpleDirective method from Paser class

"it is to check if there are any simple directives in a block directive and to parse them."

|            |            `type`             |     `name`     |                        `description`                         |
| :--------: | :---------------------------: | :------------: | :----------------------------------------------------------: |
|   return   |  std::pair<bool, Directive>   |                |                                                              |
| parameters |          std::string          |  config_path   |           config path for printing error messages            |
|            |      enum DirectiveKind       |      kind      |         current block directive kind context parsing         |
| variables  | std::vector\<Token>::iterator |  start_token   |    token iterator to save the start token before parsing     |
|            |  std::pair<bool, Directive>   | directive_pair | if the second value of the pair is valid(directive token), bool is true<br /> otherwise, it's false and second value is unusable |



### Entire code snippet for expectSimpleDirective method

```c++
std::pair<bool, Directive>	Parser::expectSimpleDirective(std::string config_path, enum DirectiveKind kind)
{
	(void)ft::sTokenTypeStrings[current_token_->type]; // for debuging
	std::vector<Token>::iterator 	start_token = current_token_;
	std::pair<bool, Directive> 		directive_pair = checkValidDirective();

	if (directive_pair.first == true) // it is a directive token
	{
		if ((directive_pair.second.directive == HTTP) ||
			(directive_pair.second.directive == SERVER) ||
			(directive_pair.second.directive == LOCATION)) // check if it's a block directive
		{
			current_token_ = start_token;
			return (std::make_pair(false, directive_pair.second)); // return false because it's not a simple directive
		}
		// check if it's a valid simple directive of particular directive kind
		if ((kind == HTTP &&
			(directive_pair.second.directive == LIMIT_EXCEPT || 
			directive_pair.second.directive == LISTEN || 
			directive_pair.second.directive == SERVER_NAME || 
			directive_pair.second.directive == RETURN || 
			directive_pair.second.directive == CGI || 
			directive_pair.second.directive == CGI_PATH)) ||
		(kind == SERVER && 
			(directive_pair.second.directive == LIMIT_EXCEPT || 
			directive_pair.second.directive == CGI || 
			directive_pair.second.directive == CGI_PATH)) ||
		(kind == LOCATION && 
			(directive_pair.second.directive == LISTEN || 
			directive_pair.second.directive == SERVER_NAME)))
		{
			std::cout << "Error: \"" << sDirectiveKindStrings[directive_pair.second.directive];
			std::cout << "\" directive is not allowed here in ";
			std::cout << config_path << ":" << current_token_->line_num << std::endl;
			current_token_ = start_token;
			return (std::make_pair(false, directive_pair.second)); // invalid directive in a wrong block directive
		}
	}
	else if (directive_pair.first == false) // it's not a directive token nor a valid directive token
	{
		if (current_token_ != end_token_ && expectToken(OPERATOR, "}").first == false)
		{
			std::cout << "Error: Unknown directive \"" << current_token_->text << "\" in ";
			std::cout << config_path << ":" << current_token_->line_num << std::endl;
		}
		current_token_ = start_token;
		return (std::make_pair(false, directive_pair.second)); // unknown directive error
	}
	return (checkValidParameterNumber(config_path, directive_pair));
}
```





#### if statement

```c++
if (directive_pair.first == true) // it is a directive token
{
	if ((directive_pair.second.directive == HTTP) ||
		(directive_pair.second.directive == SERVER) ||
		(directive_pair.second.directive == LOCATION)) // check if it's a block directive
	{
		current_token_ = start_token;
		return (std::make_pair(false, directive_pair.second)); // return false because it's not a simple directive
	}
	// check if it's a valid simple directive of particular directive kind
	if ((kind == HTTP &&
		(directive_pair.second.directive == LIMIT_EXCEPT || 
		directive_pair.second.directive == LISTEN || 
		directive_pair.second.directive == SERVER_NAME || 
		directive_pair.second.directive == RETURN || 
		directive_pair.second.directive == CGI || 
		directive_pair.second.directive == CGI_PATH)) ||
	(kind == SERVER && 
		(directive_pair.second.directive == LIMIT_EXCEPT || 
		directive_pair.second.directive == CGI || 
		directive_pair.second.directive == CGI_PATH)) ||
	(kind == LOCATION && 
		(directive_pair.second.directive == LISTEN || 
		directive_pair.second.directive == SERVER_NAME)))
	{
		std::cout << "Error: \"" << sDirectiveKindStrings[directive_pair.second.directive];
		std::cout << "\" directive is not allowed here in ";
		std::cout << config_path << ":" << current_token_->line_num << std::endl;
		current_token_ = start_token;
		return (std::make_pair(false, directive_pair.second)); // invalid directive in a wrong block directive
	}
}
```

[line 3 ~ 9]  if it is a valid directive token, it checks first if it's a block directive(http, server, location) then it returns false.

[line 11 ~ 31]  if the simple directive is valid for the particular block directive(the value of `kind` in this case), which means if the simple directive can exist in the current block directive or not, if it doesn't, it prints an error message and returns false.



#### else if statement

```c++
else if (directive_pair.first == false) // it's not a directive token nor a valid directive token
{
	if (current_token_ != end_token_ && expectToken(OPERATOR, "}").first == false)
	{
		std::cout << "Error: Unknown directive \"" << current_token_->text << "\" in ";
		std::cout << config_path << ":" << current_token_->line_num << std::endl;
	}
	current_token_ = start_token;
	return (std::make_pair(false, directive_pair.second)); // unknown directive error
}
```

[line 1 ~ 10]  if it's not a valid directive(invalid directive in a specific block directive or unknown directive). it sets the current_token_ value to start_token_ and returns false.
