---
layout: single
title: "[webserv] Parser class - expectToken method"


---

# expectToken method from Parser class

"it is to check if the current token has a valid number of parameters. it works for only simple directives."

|            |         `type`         |    `name`    |                        `description`                         |
| :--------: | :--------------------: | :----------: | :----------------------------------------------------------: |
|   return   | std::pair<bool, Token> |              |                                                              |
| parameters |     enum TokenType     |     type     |                 token type for specific one                  |
|            |   const std::string&   |     name     |            specific text value of the token type             |
| variables  |         Token          | return_token | if the current token type corresponds to the given token type and if it has a specific text value |



### Entire code snippet for expectToken method

```c++
std::pair<bool, Token>	Parser::expectToken(enum TokenType type, const std::string& name)
{
	Token return_token;
	if (current_token_ == end_token_)
		return (std::make_pair(false, return_token));
	if (current_token_->type != type)
		return (std::make_pair(false, return_token));
	if (!name.empty() && current_token_->text != name)
		return (std::make_pair(false, return_token));
	return_token = *current_token_;
	++current_token_;
	return (std::make_pair(true, return_token));
}
```

[line 4] if it's the end token, it returns false.

[line 6] if the token type doesn't match, it returns false.

[line 8] if the token text exists and it doesn't match, it returns false.