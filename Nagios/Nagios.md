

COMPONENTS

		
					Hostgroup	
					  /		\
					 /		  \
					/		   \
servicegroup -----------  service	---------- host
				  /	|	\
				/	|	 \
			       /		|	   \
		command	    contact	   contactgroup
				





CHECKS

Active Checks

Passive Checks






PLUGINS

On the host to be checked I saw the plugin in /usr/liib64/nagios/plugins/check_elastic_index_age.py.     [that python file is the plugin]





CONFIG FILES

nagios.cfg
	broker_module			-		a broker module for nagios, e.g. livestatus