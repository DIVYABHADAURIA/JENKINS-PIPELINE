import java.text.SimpleDateFormat
def sourceu = [:]
sourceu.name = 'SourceUnix'
sourceu.password = 'Unix11!'
sourceu.allowAnyHosts = true
 
def sourced = [:]
sourced.name = 'SourceDB'
sourced.user = 'oradp'
sourced.password = 'Data4u!'
sourced.allowAnyHosts = true

 
def destu = [:]
destu.name = 'TargetUnix'
destu.password = 'Unix11!'
destu.allowAnyHosts = true
 
def destd = [:]
destd.name = 'TargetDB'
destd.user = 'oradp'
destd.password = 'Data4u!'
destd.allowAnyHosts = true

def source_DB
def dest_DB
def source_DB_user
def dest_DB_user


def datep = new Date().format( 'yyyyMMdd' )
def date = datep.toString()


node{
      
    properties([ parameters([choice(choices: "MEC3", description: 'Select the source env', name: 'envs'),choice(choices:"MEC5", description: 'Select the target env', name: 'envd') ]) ])
	
	switch(envs){
		case 'MEC3':
					sourceu.host = 'incepld019.corp.amdocs.com'
                    sourceu.user = 'epcwrk3'
					sourced.host = 'incepld021.corp.amdocs.com'
					source_DB='PLDEPC2'
					source_DB_user='PLDMEC3'
					break;	
		
	}
	
	switch(envd ){

	/*	case 'MEC3':
					destu.host = 'incepld019.corp.amdocs.com'
                    destu.user = 'epcwrk3'
                    destd.host = 'incepld021.corp.amdocs.com'
					dest_DB='PLDEPC2'
					dest_DB_user='PLDMEC3'
					break; */
		case 'MEC5':
					destu.host = 'incepld019.corp.amdocs.com'
                    destu.user = 'epcwrk5'
                    destd.host = 'incepld021.corp.amdocs.com'
					dest_DB='PLDEPC2'
					dest_DB_user='PLDMEC5'
					break;

    /*    case 'MEC7':
					destu.host = 'incepld025.corp.amdocs.com'
                    destu.user = 'epcwrk7'
                    destd.host = 'incepld026.corp.amdocs.com'
					dest_DB='PLDEPC5'
					dest_DB_user='PLDMEC7'
					break;  
		case 'TEST':
					destu.host = 'incepld019.corp.amdocs.com'
                    destu.user = 'epcwrk3'
                    destd.host = 'incepld026.corp.amdocs.com'
					dest_DB='PLDEPC5'
					dest_DB_user='TEST'
					break;   */		
					
	}
	
    withCredentials([usernamePassword(credentialsId: 'sshUserAcct', passwordVariable: 'password', usernameVariable: 'userName')]) {
 
	stage('Environment Shutdown') {
	        sh 'echo "Stopping source environment"'
	        sshCommand remote: sourceu, command: "/users/gen/${sourceu.user}/JEE/mecDomain/scripts/forceStopMECServer.sh"
		def pingStatus=sshCommand remote: sourceu, failOnError: false ,command: "/users/gen/${sourceu.user}/JEE/mecDomain/scripts/pingMECServer.sh"
		println("Status of MEC Server : " + pingStatus)
		if(pingStatus == 'DOWN')
			println("MEC Server is DOWN. Proceeding with next stage...")
		else
			println("MEC Server is UP. Not proceeding with next stage...")
		assert pingStatus == 'DOWN'
	}
	stage('Source Environment backup'){
		sh 'echo "Source environment backup"'
		sshCommand remote: sourced, command: "export ORACLE_SID=${source_DB} ; export ORACLE_HOME=/oravl01/oracle/12.2.0.1 ; export PATH=$ORACLE_HOME/bin:$PATH;expdp ${source_DB_user}/${source_DB_user} directory=PLDMEC2_DUMP dumpfile=expdp_${source_DB_user}_Full_${date}%U.dmp logfile=expdp_${source_DB_user}_Full_${date}.log  reuse_dumpfiles=y parallel=10 exclude=statistics"
		def count =sshCommand remote: sourced,command: "grep ORA- /oravldp/ORACLE/PLDMEC2_DUMP/expdp_${source_DB_user}_Full_${date}.log |wc -l" 
		println(count)
		if(count=="0")
		println("SUCCESS")
		else 
        println("FAILURE")
		assert (count=="0")
		sshCommand remote: sourced, command: 'sudo /bin/chmod -R 777 /oravldp/ORACLE/*'
	}   
	    stage('Transfer of Dump to Target'){
		sh 'echo "Transferring dump file to target schema DB server "'	
		sshCommand remote: sourced, command: "/usr/bin/rsync /oravldp/ORACLE/PLDMEC2_DUMP/expdp_${source_DB_user}_Full*  ${destd.user}@${destd.host}:/oravldp/ORACLE/PLDMEC2_DUMP"
	}  
	stage('Backup of Target Environment'){ 
		sh 'echo "Backup of target DB"'
		sshCommand remote: destu, command: "/users/gen/${destu.user}/JEE/mecDomain/scripts/forceStopMECServer.sh"
		def pingStatus=sshCommand remote: destu, failOnError: false ,command: "/users/gen/${destu.user}/JEE/mecDomain/scripts/pingMECServer.sh"
		println("Status of MEC Server : " + pingStatus)
		if(pingStatus == 'DOWN')
			println("MEC Server is DOWN. Proceeding with next stage...")
		else
			println("MEC Server is UP. Not proceeding with next stage...")
		assert pingStatus == 'DOWN'
		sshCommand remote: destd, command: "export ORACLE_SID=${dest_DB} ; export ORACLE_HOME=/oravl01/oracle/12.2.0.1 ; export PATH=$ORACLE_HOME/bin:$PATH;expdp ${dest_DB_user}/${dest_DB_user} directory=PLDMEC2_DUMP dumpfile=expdp_${dest_DB_user}_Full_${date}%U.dmp  logfile=expdp_${dest_DB_user}_Full_${date}.log  reuse_dumpfiles=y parallel=10 exclude=statistics"
	    def count =sshCommand remote: destd,command: "grep ORA- /oravldp/ORACLE/PLDMEC2_DUMP/expdp_${dest_DB_user}_Full_${date}.log |wc -l" 
	    println(count)
		if(count=="0")
		println("SUCCESS")
		else 
        println("FAILURE")
		assert (count=="0")
	}   
	stage('Load to Target'){
		sh 'echo "Load"'
		sshCommand remote: destd, command: "/oravldp/oradp/drop_obj.sh ${dest_DB_user}/${dest_DB_user}@${dest_DB} ALL"
	//	sshCommand remote: destd, command: "/oravldp/oradp/drop_user.sh ${dest_DB_user}/${dest_DB_user}@${dest_DB} "
		sshCommand remote: destd, command: "export ORACLE_SID=${dest_DB} ; export ORACLE_HOME=/oravl01/oracle/12.2.0.1 ; export PATH=$ORACLE_HOME/bin:$PATH;  impdp AIM_DBA/AIM_DBA dumpfile=expdp_${source_DB_user}_Full_${date}%U.dmp logfile=IMPDP_${dest_DB_user}_Full_${date}.log directory=PLDMEC2_DUMP remap_schema=${source_DB_user}:${dest_DB_user} parallel=15",failOnError:false
		def count =sshCommand remote: destd,command: "grep ORA- /oravldp/ORACLE/PLDMEC2_DUMP/IMPDP_${dest_DB_user}_Full_${date}.log |wc -l" 
		//println(count)
		if(count=="1")
		println("SUCCESS")
		else 
        println("FAILURE")
		assert (count=="1")
		
	}
	stage('Environment startup'){
		sh 'echo "Startup"'
		sshCommand remote: destu, command: "/users/gen/${destu.user}/JEE/mecDomain/scripts/startMECServer.sh"
	}
    }
}
