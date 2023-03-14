---
layout: single
title: "[webserv] Parser class - checkValidDirective method"



---

# checkValidDirective method from Parser class

"it is to check if the current token type is DIRECTIVE."

|            |                   `type`                   |     `name`      |                        `description`                         |
| :--------: | :----------------------------------------: | :-------------: | :----------------------------------------------------------: |
|   return   |         std::pair<bool, Directive>         |                 |                                                              |
| parameters |                     -                      |        -        |                              -                               |
| variables  |          std::pair\<bool, Token>           |   token_pair    | if the current token is a directive kind, bool is true and the second value of the pair refers to the token(its type is DIRECTIVE)<br />otherwise, it's false and second value is unusable |
|            | std::map<std::string, Directive>::iterator | found_directive | to filter if the token text is a valid directive from the directives map |



### Entire code snippet for checkValidDirective method

```c++
std::pair<bool, Directive>	Parser::checkValidDirective()
{
  std::pair<bool, Token>                      token_pair = expectToken(DIRECTIVE);
  std::map<std::string, Directive>::iterator  found_directive;

  if (!token_pair.first) // it's not a directive token
  {
    return (std::make_pair(false, directives_.begin()->second)); 
  }
  found_directive = directives_.find(token_pair.second.text);
  // check if the directive token is valid
  if (found_directive == directives_.end())
  {
    --current_token_;
    return (std::make_pair(false, directives_.begin()->second)); 
  }
  return (std::make_pair(true, found_directive->second)); 
}
```

[line 4 ~ 8]  if the current token is not type of DIRECTIVE, it returns false and the second value of pair is not usable.

[line 9 ~ 16]  it checks if the current token text is valid from the directives map.
