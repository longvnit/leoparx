./certbot-auto certonly -d abc.com -d www.abc.com --webroot -w /var/www/html --non-interactive --agree-tos -m admin@abc.com


USE_NEW_SET_PASSWORD=1
	INDENTIFIED_WITH_STRING="IDENTIFIED WITH mysql_native_password"
	#for MySQL 8.0, SET PASSWORD doesn't specify PASSWORD()
	if [ "${MYSQL_INST_OPT}" = "mysql" ]; then
		if [ "${MYSQLV}" = "5.0" ] || [ "${MYSQLV}" = "5.1" ] || [ "${MYSQLV}" = "5.5" ] || [ "${MYSQLV}" = "5.6" ]; then
			USE_NEW_SET_PASSWORD=0
			INDENTIFIED_WITH_STRING="IDENTIFIED"
		fi
	else
		if [ "`has_mariadb`" = "1" ]; then
			if [ "${MYSQLV}" = "10.3" ] || [ "${MYSQLV}" = "10.4" ]; then
				USE_NEW_SET_PASSWORD=1
			else
				USE_NEW_SET_PASSWORD=0
			fi
			INDENTIFIED_WITH_STRING="IDENTIFIED"
		else
			USE_NEW_SET_PASSWORD=0
			INDENTIFIED_WITH_STRING="IDENTIFIED"
		fi
	fi

	if [ "${HAVE_SQL_USER}" != "1" ]; then
		echo "Creating User: CREATE USER '${SQL_USER}'@'${SQL_HOST}' ${INDENTIFIED_WITH_STRING} BY '${SQL_PASS}';"
		mysql --defaults-extra-file=${DA_MY_CNF} -e "CREATE USER '${SQL_USER}'@'${SQL_HOST}' ${INDENTIFIED_WITH_STRING} BY '${SQL_PASS}';" --host=${MYSQLHOST} 2>&1
	fi
	
	echo "Granting access: GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,LOCK TABLES,INDEX,REFERENCES ON ${SQL_DB}.* TO '${SQL_USER}'@'${SQL_HOST}';"
	mysql --defaults-extra-file=${DA_MY_CNF} -e "GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,LOCK TABLES,INDEX,REFERENCES ON ${SQL_DB}.* TO '${SQL_USER}'@'${SQL_HOST}';" --host=${MYSQLHOST} 2>&1
	
	if [ "${USE_NEW_SET_PASSWORD}" = "1" ]; then
		echo "Setting password: ALTER USER '${SQL_USER}'@'${SQL_HOST}' ${INDENTIFIED_WITH_STRING} BY '${SQL_PASS}';"
		mysql --defaults-extra-file=${DA_MY_CNF} -e "ALTER USER '${SQL_USER}'@'${SQL_HOST}' ${INDENTIFIED_WITH_STRING} BY '${SQL_PASS}';" --host=${MYSQLHOST} 2>&1
	else
		echo "Setting password: SET PASSWORD FOR '${SQL_USER}'@'${SQL_HOST}' = PASSWORD('${SQL_PASS}');"
		mysql --defaults-extra-file=${DA_MY_CNF} -e "SET PASSWORD FOR '${SQL_USER}'@'${SQL_HOST}' = PASSWORD('${SQL_PASS}');" --host=${MYSQLHOST} 2>&1
	fi
	

    mysql_install_db --user=mysql --datadir=/var/lib/mysql --auth-root-authentication-method=normal