---
layout: single
title: "[webserv] Parser class - checkValidParameterNumber method"

---

# checkValidParameterNumber method from Parser class

"it is to check if the current token has a valid number of parameters. it works for only simple directives."

|            |           `type`            |        `name`        |                        `description`                         |
| :--------: | :-------------------------: | :------------------: | :----------------------------------------------------------: |
|   return   | std::pair<bool, Directive>  |                      |                                                              |
| parameters |         std::string         |     config_path      |           config path for printing error messages            |
|            | std::pair<bool, Directive>& |    directive_pair    |          a reference to a pair of bool, directive            |
| variables  |   std::pair\<bool, Token>   | parameter_token_pair | if the current token type is PARAMETER then bool is true and the second value of the pair refers to the token<br />otherwise, it's false and second value is unusable |



### Entire code snippet for checkValidDirective method

```c++
std::pair<bool, Directive> Parser::checkValidParameterNumber(std::string config_path, std::pair<bool, Directive>& directive_pair)
{
	std::pair<bool, Token> 		parameter_token_pair = expectToken(PARAMETER);

 	if (parameter_token_pair.first == true) // if the token type is PARAMETER
	{
		if ((directive_pair.second.directive == SERVER_NAME) ||
				(directive_pair.second.directive == LIMIT_EXCEPT) ||
				(directive_pair.second.directive == INDEX) ||
				(directive_pair.second.directive == ERROR_PAGE) ||
				(directive_pair.second.directive == RETURN))
		{
			while (parameter_token_pair.first == true)
			{
				directive_pair.second.line_num = current_token_->line_num;
				directive_pair.second.parameters.push_back(parameter_token_pair.second.text);
				parameter_token_pair = expectToken(PARAMETER);
			}
		}
		else
		{
			directive_pair.second.line_num = current_token_->line_num;
			directive_pair.second.parameters.push_back(parameter_token_pair.second.text);
		}
	}
	else
	{
		std::cout << "Error: invalid number of arguments in \"" << sDirectiveKindStrings[directive_pair.second.directive];
		std::cout << "\" directive in " << config_path << ":" << current_token_->line_num << std::endl;
		return (std::make_pair(false, directive_pair.second));
	}
	parameter_token_pair = expectToken(PARAMETER);
	if (parameter_token_pair.first == true)
	{
		std::cout << "Error: invalid parameter \"" << parameter_token_pair.second.text << "\" in ";
		std::cout << config_path << ":" << current_token_->line_num << std::endl;
		return (std::make_pair(false, directive_pair.second));
	}
	std::pair<bool, Token> token_pair = expectToken(OPERATOR, ";");
	if (token_pair.first == false)
	{
		std::cout << "Error: directive \"" << sDirectiveKindStrings[directive_pair.second.directive];
		std::cout << "\" is not terminated by \";\" in ";
		std::cout << config_path << ":" << (current_token_ - 1)->line_num << std::endl;
		return (std::make_pair(false, directive_pair.second));
	}
	return (std::make_pair(true, directive_pair.second));
}
```



#### if-else statement from the code snippet

```c++
if (parameter_token_pair.first == true) // if the token type is PARAMETER
{
	if ((directive_pair.second.directive == SERVER_NAME) ||
			(directive_pair.second.directive == LIMIT_EXCEPT) ||
			(directive_pair.second.directive == INDEX) ||
			(directive_pair.second.directive == ERROR_PAGE) ||
			(directive_pair.second.directive == RETURN))
	{
		while (parameter_token_pair.first == true)
		{
			directive_pair.second.line_num = current_token_->line_num;
			directive_pair.second.parameters.push_back(parameter_token_pair.second.text);
			parameter_token_pair = expectToken(PARAMETER);
		}
	}
	else
	{
		directive_pair.second.line_num = current_token_->line_num;
		directive_pair.second.parameters.push_back(parameter_token_pair.second.text);
	}
}
else
{
	std::cout << "Error: invalid number of arguments in \"" << sDirectiveKindStrings[directive_pair.second.directive];
	std::cout << "\" directive in " << config_path << ":" << current_token_->line_num << std::endl;
	return (std::make_pair(false, directive_pair.second));
}
```

[line 3 ~ 6]  directives that can have multiple parameters are SERVER_NAME, LIMIT_EXCEPT, INDEX, ERROR_PAGE, RETURN and it checks if the current directive is one of them.

[line 7 ~ 14] in the while loop, it sets the line_num value of the second directive_pair for printing error messages and parses every parameter value of the current directive.

[line  16 ~ 20]  if it is not a directive that can have multiple parameters, it just sets the parameter value.

[line 22 ~ 27]  if the current token type is not PARAMETER, it prints an error message.





#### last part of the code snippet

```c++
parameter_token_pair = expectToken(PARAMETER);
if (parameter_token_pair.first == true)
{
	std::cout << "Error: invalid parameter \"" << parameter_token_pair.second.text << "\" in ";
	std::cout << config_path << ":" << current_token_->line_num << std::endl;
	return (std::make_pair(false, directive_pair.second));
}
std::pair<bool, Token> token_pair = expectToken(OPERATOR, ";");
if (token_pair.first == false)
{
	std::cout << "Error: directive \"" << sDirectiveKindStrings[directive_pair.second.directive];
	std::cout << "\" is not terminated by \";\" in ";
	std::cout << config_path << ":" << (current_token_ - 1)->line_num << std::endl;
	return (std::make_pair(false, directive_pair.second));
}
return (std::make_pair(true, directive_pair.second));
```

[line 1 ~ 7] after finishing setting all parameters of the directive, if it has another parameter, it prints an error message.

[line 8 ~ 14] it verifies if the current line ends with `;` 
