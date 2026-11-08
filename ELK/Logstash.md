


LOGSTASH


Pipeline 		
- receiving, processing and shipping events
- it is an orchestration of plugins
- think of a pipeline as input + (filter) + output


Codecs
these change the data representation of events
e.g.  parse incoming data as json



Data Sources



Destinations




Execution Model of Logstash



			Input	/codec					----------------	batch  -->		filters		outputs/codec
										|		   |			{worker A}
										|		   |
			Input	/codec					 work queue 	batch  -->		filters		outputs/codec
										|		  |			{worker B}
										|		  |			
			Input/codec					|_________|	batch  -->		filters		outputs/codec
															{worker C}



STARTUP OPTIONS

--config.reload.automatic



OPERATORS

equality				==, !=, >=, <=, >, <
pattern match			=~, !~
inclusion				in, not in				[ to check if a string contains some value or a list contains some element ]
boolean 				and, or
unary				!					[ negates the EXPR it precedes ]
parenthesis			( )					group expressions



SPRINTF

a way of specifying a format for files, indexes, etc.
file => “/tmp/%{type}_%{+yyyy_MM_dd}.log"
index => "%{type}-%{+YYYY.MM.dd}"




file PLUGIN

path globs
/path/to/*.log
/path/to/**/*.log
/path/to/{nginx,apache}/*.log



grok PLUGIN

%{SYNTAX:SEMANTIC:[data type]}


