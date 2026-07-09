
BASICS

literals - string is "string", integer is 42, float is 42.33, list is
\[1, 2, 3\], tuple is a list that cant be modified, dictionary {key1:
value1, key2: value2}, boolean
math operators - + - % \*
comparison - ==, !=, \>=, \<=
logical operators - and, or, not, in, not in,
group expressions - ()



DELIMITERS

{%\...%} - for Statements
{{\...}} - for Expressions
{#..#} - for Comments



IF-ELSE

{% if \<condition\> %}
do something
{% endif %}



FOR LOOP

{% for \<item\> in \<list\> %}
do something with "{{ \<item\> }}"
{% endfor %}

{% for host in groups\[\'lamp_www\'\] %} \<--- groups is a magic
variable in ansible
"{{ host \| upper}}" \<- produces a string with the hostname in
capitals
{% endfor %}




FILTERS

{{ \<var_name\> \| striptags \| title }} - will remove HTML tags from
\<var_name\> and title-case the output
{{ \<list_name\> \| join(', ') }} - will join the list elements with
comma space

List of builtin filters -
https://jinja.palletsprojects.com/en/3.0.x/templates/#builtin-filters



TESTS

{% if \<var_name\> is equalto 3 %}
{% if \<var_name\> is defined %}

List of builtin tests -
https://jinja.palletsprojects.com/en/3.0.x/templates/#builtin-tests



ESCAPING

{% raw %}
jinja syntax that would normally be interpreted by jinja
{% endraw %}

{% raw -%} - with minus sign, this will clean all spaces and newlines
preceding the first character of your raw data
