---
layout: single
title: "[webserv] Parser class - expectHttpContext method"



---

# expectHttpContext method from Parser class

"it is to check if the current token is a valid http directive."

|            |            `type`            |     `name`     |                        `description`                         |
| :--------: | :--------------------------: | :------------: | :----------------------------------------------------------: |
|   return   |  std::pair<bool, Directive>  |                |                                                              |
| parameters |              -               |       -        |                              -                               |
| variables  | std::vector<Token>::iterator |  parse_start   | to save the start token iterator before parsing, it's for when an error occurs |
|            |  std::pair<bool, Directive>  | directive_pair | if the second value of the pair is valid(directive token), bool is true<br /> otherwise, it's false and second value is unusable |



### Entire code snippet for expectHttpContext method

```c++
std::pair<bool, Directive>	Parser::expectHttpContext()
{
	std::vector<Token>::iterator	parse_start = current_token_;
	std::pair<bool, Directive>		directive_pair = checkValidDirective();

	if (directive_pair.first == false) // when it doesn't start with a directive
	{
		if (current_token_ == start_token_)
			std::cout << "Error: First directive should be http block.\n";
		current_token_ = parse_start;
		return (std::make_pair(false, directive_pair.second));
	}
	if (directive_pair.second.directive != HTTP) // when the directive isn't http
	{
		current_token_ = parse_start;
		std::cout << "Error: It should start with http context but the directive is ";
		std::cout << directive_pair.second.name << "\n";
		return (std::make_pair(false, directive_pair.second));
	}
	// until now it starts with http directive and there isn't any parameter
	if (expectToken(OPERATOR, "{").first == false) // to check there isn't any parameter of http directive and encloses with {
	{
		std::cout << "Error: Http context can't have any parameter.\n";
		return (std::make_pair(false, directive_pair.second));
	}
	return (std::make_pair(true, directive_pair.second));
}
```

[line 6 ~ 12]  if the current token type is not DIRECTIVE, it returns false.

[line 13 ~ 19]  if the current token type is DIRECTIVE but not http, it returns false.

[line 20 ~ 24]  the current token type is DIRECTIVE and it's http, in this case, it checks if http directive has no parameters and start with `{`.

