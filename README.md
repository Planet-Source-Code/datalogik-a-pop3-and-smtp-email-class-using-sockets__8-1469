<div align="center">

## A POP3 and SMTP email class using sockets\.


</div>

### Description

This is an email class that will let you check your email, delete email from server, and send email without any built-in commands such as mail() and other imap functions.
 
### More Info
 
When testing the example code at the bottom, remove the /* and */ comments to enable whichever section.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[datalogik](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/datalogik.md)
**Level**          |Advanced
**User Rating**    |4.9 (34 globes from 7 users)
**Compatibility**  |PHP 3\.0, PHP 4\.0
**Category**       |[Internet/ Browsers/ HTML](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/internet-browsers-html__8-9.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/datalogik-a-pop3-and-smtp-email-class-using-sockets__8-1469/archive/master.zip)

### API Declarations

Copyright (c) 2004 Jonathan Anders


### Source Code

```
<?php
	/* A POP3 and SMTP email class using sockets. An exerpt from phpMailIt WebMail Client.
	 By: Jonathan Anders
	 Website: http://www.datalogik.org
	 Email: datalogik@datalogik.org */
	set_time_limit(0);
	class phpMailIt
	{
		// Private variables
		var $_serv;
		var $_port;
		var $_usern;
		var $_pass;
		var $_to;
		var $_subj;
		var $_body;
		var $_socket;
		// Toggle this if you want to see what the script does in the background, it will screw up the look of the webmail though.
		var $_debug	= 1;
		// Do not edit these. These are here to identify the mailing program.
		var $_app_name	= 'phpMailIt';
		var $_app_desc	= 'Web E-Mail';
		var $_app_ver 	= '1.0';
		// POP Functions
		function pop3_init ($server, $port, $username, $password)
		{
			// Specify variables, obviously.
			$this->_serv = $server;
			$this->_port = (int)$port;
			$this->_usern = $username;
			$this->_pass = $password;
		}
		function pop3_connect ()
		{
			// Check to see if the hostname was given.
			if ($this->_serv == "")
				$this->mail_output("Hostname was not specified.");
			// Check for debuggy thingy, theres loads of these.
			if ($this->_debug)
				$this->mail_output("Connecting to ".$this->_serv." ...");
			// Create the socket on whatever port is specified.
			$this->_socket = fsockopen($this->_serv,$this->_port);
		}
		function pop3_login ()
		{
			// Retrieve data from the server, not getting mail yet.
			$_mail_get = $this->mail_get();
			// Send the 'USER' command to authenticate with the POP3 server.
			if ($this->mail_write("USER $this->_usern") == 0)
				return("Could not send the USER command");
			// Again, retrieving data from server.
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			// Split the data into parameters
			$parse = explode(" ", $_mail_get);
			// Problems with the 'USER' command have occured.
			if ($parse[0] != "+OK")
				return("User error: authentication with USER command.");
			// Send the 'PASS' command to authenticate with the POP3 server, last of 2 parts.
			if ($this->mail_write("PASS $this->_pass") == 0)
				return("Could not send the PASS command");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$parse = explode(" ", $_mail_get);
			if ($parse[0] != "+OK")
				return("User error: authentication with PASS command.");
		}
		function pop3_check_messages()
		{
			// Send the 'STAT' command to check for new messages
			if ($this->mail_write("STAT") == 0)
				return("Could not send the STAT command");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$parse = explode(" ", $_mail_get);
			if ($parse[0] != "+OK")
				return("Error: issues with STAT.");
			if ($this->_debug)
				$this->mail_output($parse[1]);
			if ($parse[1] == 0)
				if ($this->_debug)
					$this->mail_output("No new mail");
			// For loop, retrieving however many messages from the server.
			for ($x = 1; $x <= $parse[1]; $x++)
			{
				// Null the values
				$pop3_from[$x] = "";
				$pop3_to[$x] = "";
				$pop3_subject[$x] = "";
				$pop3_date[$x] = "";
				$pop3_received[$x] = "";
				$pop3_xmailer[$x] = "";
				$pop3_body[$x] = "";
				// Send the 'RETR' command.
				if ($this->mail_write("RETR $x") == 0)
					return("Could not send the RETR command");
				// While loop, make sure it downloads the entire email content.
				while (($_mail_get = $this->mail_get()) != ".")
				{
					if ($this->_debug)
						$this->mail_output($_mail_get);
					$_mail_get_parse = explode(" ", $_mail_get);
					// This was chosen to format the data so it can be read by the user.
					switch (strtolower($_mail_get_parse[0]))
					{
						case "from:":
							$pop3_from[$x] = $_mail_get_parse[1];
							break;
						case "to:":
							$pop3_to[$x] = substr($_mail_get,3,strlen($_mail_get)-3);
							break;
						case "subject:":
							$pop3_subject[$x] = substr($_mail_get,8,strlen($_mail_get)-8);
							break;
						case "date:":
							$pop3_date[$x] = substr($_mail_get,5,strlen($_mail_get)-5);
							break;
						case "received:":
							$pop3_received[$x] = substr($_mail_get,9,strlen($_mail_get)-9);
							break;
						case "x-mailer:":
							$pop3_xmailer[$x] = substr($_mail_get,9,strlen($_mail_get)-9);
							break;
						case "+ok":
							break;
						default:
							$pop3_body[$x] .= $_mail_get . "\n\r";
					}
				}
				// The output of the emails.
				if ($this->_debug)
				{
					$this->mail_output("To: ". $pop3_to[$x]);
					$this->mail_output("From: ". $pop3_from[$x]);
					$this->mail_output("Subject: ". $pop3_subject[$x]);
					$this->mail_output("Date: ". $pop3_date[$x]);
					$this->mail_output("Received: ". $pop3_received[$x]);
					$this->mail_output("X-Mailer: ". $pop3_xmailer[$x]);
					$this->mail_output("Body: <PRE>". $pop3_body[$x] . "</PRE>");
					$this->mail_output("<br><br><br><b>LINE BREAK</b><br><br><br>");
				}
			}
		}
		function pop3_delete_messages ($message_numbers)
		{
			for ($x = 0; $message_numbers[$x]; $x++)
			{
				// Send the 'DELE' command.
				if ($this->mail_write("DELE $message_numbers[$x]") == 0)
					return("Could not send the DELE command");
				$_mail_get = $this->mail_get();
				if ($this->_debug)
					$this->mail_output($_mail_get);
			}
		}
		// SMTP Functions
		function smtp_init ($server, $port)
		{
			// Specify variables, obviously, not many here.
			$this->_serv = $server;
			$this->_port = (int)$port;
		}
		function smtp_connect ()
		{
			// Check to see if the hostname was given.
			if ($this->_serv == "")
				$this->mail_output("Hostname was not specified.");
			// Check for debuggy thingy, theres loads of these.
			if ($this->_debug)
				$this->mail_output("Connecting to ".$this->_serv." ...");
			// Create the socket on whatever port is specified.
			$this->_socket = fsockopen($this->_serv,$this->_port);
			// Tell the script to login.
			//$this->smtp_send_email ($this->_to, $this->_subj, $this->_body);
		}
		function smtp_hand_shake ()
		{
			// Send the 'HELO' command to the server to let them know we mean business.
			$helo = 'HELO $' . 'host';
			if ($this->mail_write($helo) == 0)
				return("Could not send the HELO command");
			// Again, retrieving data from server.
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			// Split the data into parameters
			$parse = explode(" ", $_mail_get);
			// Problems with the 'HELO' command have occured.
			if (($parse[0] != "250") or ($parse[0] != "220"))
				return ("Could not continue with hand shake");
		}
		function smtp_send_email ($from, $to, $subject, $body)
		{
			// Start compiling the email.
			if ($this->mail_write("MAIL FROM: $from") == 0)
				return("Could not send the MAIL FROM command");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$parse = explode(" ", $_mail_get);
			if ($parse[0] != "250")
				return("User error: problems with sending email.");
			if ($this->mail_write("RCPT TO: $to") == 0)
				return("Could not send the RCPT command");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$parse = explode(" ", $_mail_get);
			if ($parse[0] != "250")
				return("User error: problems with sending email.");
			if ($data_already_sent == 0)
				if ($this->mail_write("DATA") == 0)
					return("Could not send the DATA command");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$parse = explode(" ", $_mail_get);
			if (($parse[0] == "250") or ($parse[0] == "220") or ($parse[0] == "354"))
			{
				$this->mail_write("X-Mailer: $this->_app_name - $this->_app_desc Version: $this->_app_ver");
				$this->mail_write("FROM: $from");
				$this->mail_write("TO: $to");
				$this->mail_write("Subject: $subject");
				$this->mail_write("$body");
				$this->mail_write(".");
			}
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
			$data_already_sent = 1;
		}
		// Misc Functions
		function mail_output ($print)
		{
			echo "$print<p>\n";
			return;
		}
		function mail_quit ()
		{
			$this->mail_write("QUIT");
			$_mail_get = $this->mail_get();
			if ($this->_debug)
				$this->mail_output($_mail_get);
		}
		function mail_write ($data)
		{
			if ($this->_debug)
				$this->mail_output ($data);
			// Sending stuff to the server right here.
			return(fputs ($this->_socket, $data . "\r\n"));
		}
		function mail_get($socket)
		{
			// Retrieving stuff from the server for the first 100 bytes.
			for ($line="";;)
			{
				if (feof($this->_socket))
					return(0);
				$line .= fgets($this->_socket,100);
				$length = strlen($line);
				if (($length >= 2) && (substr($line,$length-2,2) == "\r\n"))
				{
					$line = substr($line,0,$length-2);
					return($line);
				}
			}
		}
	}
	//Check Email (to use this the /* and */ comments)
	/*
	// Create a new mail object.
	$array[0] = new phpMailIt;
	// Send the pop3_init command with the server, port, username, and password.
	$array[0]->pop3_init ("localhost", 110, "username", "password");
	// Send the pop3_connect command to have the script connect to the server.
	$array[0]->pop3_connect ();
	// Logging into the server, sending the USER and PASS commands.
	$array[0]->pop3_login ();
	// Check for new messages.
	$array[0]->pop3_check_messages ();
	// Tell the script to disconnect.
	$array[0]->mail_quit ();
	*/
	//Send Email (to use this the /* and */ comments)
	// Create a new mail object.
	$array[0] = new phpMailIt;
	// Send the smtp_init command with the server and port.
	$array[0]->smtp_init ("localhost", 25);
	// Send the smtp_connect command to have the script connect to the server.
	$array[0]->smtp_connect ();
	// Tell the server that you wish to send an email and wait for a response.
	$array[0]->smtp_hand_shake ();
	// Send all of the needed commands to send an email.
	$array[0]->smtp_send_email ("email@from.adddress.com", "email@to.address.com", "Subject here", "Body of email here");
	// Tell the script to disconnect.
	$array[0]->mail_quit ();
	//Delete Email Message (to use this the /* and */ comments)
	/*
	// Provide an array of messages that you want deleted by number. eg: you want messages 1, 4, 5, and 8 deleted.
	$test = array('1','4', '5', '8');
	// Create a new mail object.
	$array[0] = new phpMailIt;
	// Send the pop3_init command with the server, port, username, and password.
	$array[0]->pop3_init ("localhost", 110, "username", "password");
	// Send the pop3_connect command to have the script connect to the server.
	$array[0]->pop3_connect ();
	// Logging into the server, sending the USER and PASS commands.
	$array[0]->pop3_login ();
	// Tell the server to delete all of the message numbers in the array() that was provided.
	$array[0]->pop3_delete_messages ($test);
	// Tell the script to disconnect.
	$array[0]->mail_quit ();
	*/
?>
```

