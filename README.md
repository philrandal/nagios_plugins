This is a collection of plugins for Icinga, Nagios, or standalone use.

All code is licensed under GPL v2 except `check_zone_rrsig_expiration` which is licensed as described in the source file.

All code written by Michael Newton, with the exception of:

* `check_mysql_slave_status` by Claudio Kuenzler
* `check_powerconnect` by Jason Leonard
* `check_zone_rrsig_expiration` by The Measurement Factory Inc.

Sample Icinga configuration to use these commands is provided here. Simply configure hosts with the appropriate variables.
```
#
# Commands
#
object CheckCommand "radiuscheck" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_radclient" ]

	arguments = {
		"-f" = {
			required = true
			value = "$radius_type$"
		}
		"-H" = {
			value = "$radius_host$"
		}
		"-p" = {
			value = "$radius_port$"
		}
		"-s" = {
			value = "$radius_secret$"
		}
		"-a" = {
			value = "$radius_avpairs$"
			repeat_key = true
		}
		"-A" = {
			value = "$radius_binpairs$"
			repeat_key = true
		}
		"-F" = {
			set_if = "$radius_perfdata$"
		}
		"-c" = {
			value = "$radius_critical$"
		}
		"-w" = {
			value = "$radius_warning$"
		}
	}
	vars.radius_host = "$address$"
	vars.radius_type = "auth"
	vars.radius_perfdata = true
}

object CheckCommand "check-powerconnect" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_powerconnect" ]

	arguments = {
		"-H" = {
			value = "$powerconnect_host$"
			description = "report on HOST"
		}
		"-C" = {
			value = "$powerconnect_community$"
			description = "SNMP community to connect with"
		}
		"-t" = {
			value = "$powerconnect_check$"
			description = "check to perform: assets, uptime, ports, port, health, temps, fans, psus"
		}
		"-p" = {
			value = "$powerconnect_ports$"
			description = "ports to check when check type is port or ports"
		}
		"-w" = {
			value = "$powerconnect_warn$"
			description = "temperature warning threshold (degrees)"
		}
		"-c" = {
			value = "$powerconnect_crit$"
			description = "temperature critical threshold (degrees)"
		}
	}
	vars.powerconnect_host = "$address$"
	vars.powerconnect_community = "$host.vars.snmp_community$"
	vars.powerconnect_check = "health"
}

template CheckCommand "snmp-command-common" {
	import "plugin-check-command"
	import "ipv4-or-ipv6"

	arguments = {
		"-H" = {
			description = "Hostname"
			value = "$address$"
			required = true
		}

		"-p" = {
			description = "Port"
			value = "$snmp_port$"
		}

		"-C" = {
			description = "SNMP version 1/2c community (default public)"
			value = "$snmp_community$"
		}

		"-2" = {
			description = "Use SNMP version 2 (default 1)"
			set_if = "$snmp_v2$"
		}

		"-3" = {
			description = "Use SNMP version 3 (default 1)"
			set_if = "$snmp_v3$"
		}

		"-l" = {
			description = "SNMP v3 username"
			value = "$snmp_login$"
			set_if = "$snmp_v3$"
		}

		"-L" = {
			description = "SNMP v3 authentication protocol (default md5)"
			value = "$snmp_authprotocol$"
			set_if = "$host.vars.snmp_v3_use_authprotocol$"
		}

		"-x" = {
			description = "SNMP v3 authentication password"
			value = "$snmp_authpassword$"
			set_if = "$snmp_v3$"
		}

		"-X" = {
			description = "SNMP v3 privacy password"
			value = "$snmp_privpassword$"
			set_if = "$snmp_v3_use_privpass$"
		}

		"-c" = {
			description = "Critical level"
			value = "$snmp_crit$"
		}

		"-w" = {
			description = "Warning level"
			value = "$snmp_warn$"
		}

		"-f" = {
			description = "Output performance data"
			set_if = "$snmp_perf$"
		}
	}
}

object CheckCommand "check-asa" {
	import "snmp-command-common"
	command = [ PluginContribDir + "/nagios_plugins/check_asa" ]

	arguments += {
		"-M" = {
			description = "Mode: failover, cpu, temp"
			value = "$asa_checkmode$"
			required = true
		}
	}
	vars.snmp_address = "$check_address$"
	vars.snmp_port = "161"
	vars.snmp_nocrypt = true
	vars.snmp_community = "public"
	vars.snmp_v2 = false
	vars.snmp_v3 = false
	vars.snmp_login = "snmpuser"
	vars.snmp_v3_use_privpass = false
	vars.snmp_v3_use_authprotocol = false
	vars.snmp_authprotocol = "md5,des"
	vars.snmp_perf = true
	vars.snmp_timeout = "5"
	vars.asa_checkmode = "$host.vars.asa_checkmode$"
}

object CheckCommand "dell-md3200" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_dell_md3200" ]

	arguments = {
		"--c1" = "$check_address$"
		"--c2" = "$backup$"
		"--no-ping-check" = {
			set_if = "$noping$"
		}
		"--md3000" = {
			set_if = "$oldfirmware$"
		}
		"--stats" = {
			set_if = "$stats$"
		}
		"--health" = {
			set_if = "$health$"
		}
		"--vd" = {
			set_if = "$vd$"
		}
	}
	vars.backup = "$host.vars.dell_md3200_backup$"
	vars.noping = "$host.vars.dell_md3200_noping$"
	vars.oldfirmware = true
	vars.stats = "$host.vars.dell_md3200_stats$"
	vars.health = "$host.vars.dell_md3200_health$"
	vars.vd = "$host.vars.dell_md3200_vd$"
}

object CheckCommand "mysql-slave" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_mysql_slavestatus" ]

	arguments = {
		"-H" = {
			description = "Hostname"
			value = "$address$"
			required = true
		}

		"-P" = {
			description = "Port"
			value = "$mysql_repl_port$"
		}

		"-u" = {
			description = "User name"
			value = "$mysql_repl_user$"
		}

		"-p" = {
			description = "Password"
			value = "$mysql_repl_password$"
		}

		"-s" = {
			description = "Connection name"
			value = "$mysql_repl_connection$"
		}

		"-c" = {
			description = "Critical level for replication delay"
			value = "$mysql_repl_crit$"
		}

		"-w" = {
			description = "Warning level for replication delay"
			value = "$mysql_repl_warn$"
		}

		"-f" = {
			set_if = "$mysql_repl_perf$"
		}
	}
	vars.mysql_port = "3306"
}

object CheckCommand "dnssec-expiry" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_zone_rrsig_expiration" ]
	arguments = {
		"-Z" = "$dns_zone$"
		"-W" = "20"
		"-C" = "10"
	}
}

object CheckCommand "digitalocean" {
	import "plugin-check-command"
	command = [ PluginContribDir + "/nagios_plugins/check_digitalocean" ]
	arguments = {
		"-t" = "$doctl_api_password$"
		"-w" = "$doctl_warning$"
		"-c" = "$doctl_critical$"
		"-f" = "$doctl_perfdata$"
	}
}

object CheckCommand "apcload" {
	import "snmp-command-common"
	command = [ PluginContribDir + "/nagios_plugins/check_apc_pdu" ]
}

#
# Services
#
apply Service "powerconnect" for (pc in host.vars.powerconnect_checks) {
        import "generic-service"
        check_command = "check-powerconnect"
        vars.powerconnect_check = pc
}

apply Service "asa-failover" {
        import "generic-service"
        check_command = "check-asa"
        vars.asa_checkmode = "failover"
        assign where ((host.vars.snmp_v2 && host.vars.snmp_community) || (host.vars.snmp_v3 && host.vars.snmp_login)) && host.vars.os == "asa"
}

apply Service "asa-temperature" {
        import "generic-service"
        check_command = "check-asa"
        vars.asa_checkmode = "temp"
        assign where ((host.vars.snmp_v2 && host.vars.snmp_community) || (host.vars.snmp_v3 && host.vars.snmp_login)) && host.vars.os == "asa"
}

apply Service "asa-cpu" {
        import "generic-service"
        check_command = "check-asa"
        vars.asa_checkmode = "cpu"
        vars.snmp_warn = "70"
        vars.snmp_crit = "90"
        assign where ((host.vars.snmp_v2 && host.vars.snmp_community) || (host.vars.snmp_v3 && host.vars.snmp_login)) && host.vars.os == "asa"
}
apply Service "dell-md3200-check" {
        import "generic-service"
        check_command = "dell-md3200"
        check_interval = 10m
        assign where host.vars.dell_md3200 == true
}

apply Service "mysql-slave-check" {
        import "generic-service"
        check_command = "mysql-slave"
        assign where host.vars.mysql_repl_user && host.vars.mysql_repl_password
}

apply Service "dnssec-expiry-" for (zone in host.vars.dns_zones) {
        import "generic-service"
        check_command = "dnssec-expiry"
        check_interval = 4h
        vars.dns_zone = zone
}

apply Service "digitalocean-check" {
        import "generic-service"
        check_command = "digitalocean"
        check_interval = 1h
        assign where host.vars.doctl_api_password
}

apply Service "apc-load-check" {
        import "generic-service"
        check_command = "apcload"
        check_interval = 3m
        assign where host.vars.apc
}

apply Service "radius-check" {
        import "generic-service"
        check_command = "radiuscheck"
        check_interval = 5m
        assign where host.vars.radius_secret
}
```
