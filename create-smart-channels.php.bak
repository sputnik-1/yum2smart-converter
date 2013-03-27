#!/usr/bin/php

<?php

/**
 *********************************************************************
 * create-smart-channels.php
 * Yum Repo to Smart PM channel convertor script.
 *
 * Copyright Keith Roberts ~ keith@karsites.net
 * Please see BSD-LICENSE for full details
 *
 *********************************************************************
 */

 if ($argc < 3 || in_array($argv[1], array('--help', '-help', '-h', '-?')))
{

$message = <<<HDOC
  Usage:

  $argv[0] <release-version> <basearch> [d]

  <release-version> is a required parameter for the Fedora version number.

  <basearch> is a required parameter for the target OS platform.
  
  [d] is an optional parameter to turn on debugging output.
  
HDOC;

echo $message;
exit("\n");
}


$RELEASE_VERSION = $argv[1];
$BASE_ARCH = $argv[2];


if (isset($argv[3]) && 'd' == $argv[3]) {
    $DEBUG = true;
}
else {
	$DEBUG = false;
}


# Path to yum repo dir - should be CWD.
$YUM_REPOS_DIR = './';

# Path to Smart Channels dir.
$SMART_CHANNELS_DIR = '/etc/smart/channels';


if ($handle = opendir($YUM_REPOS_DIR)) {
    if ($DEBUG) {
    	echo "Directory handle: $handle\n\n";
	    echo "Contents of CWD:\n\n";
    }

    /* This is the correct way to loop over the directory. */
    while (false !== ($dir_content = readdir($handle))) {
        if ($DEBUG) {
    		echo "$dir_content\n";
        }
        
        # Filter the Yum repo filenames into an array.
        # Does the filename end with .repo?
        if (strpos($dir_content, '.repo')) {
            $yum_repo_file[] = $dir_content;
	    }
    }
    closedir($handle);
}


# Sort the $yum_repo_file array into alphabetical order.
sort($yum_repo_file);



//------------------DEBUG code----------------------//
# Check the contents of $yum_repo_file[].
if ($DEBUG) {
	echo "\nContents of \$yum_repo_file[] \n\n";
	while (list($key, $value) = each($yum_repo_file)) {
    	echo "$key: $value \n";
	}
}
//------------------DEBUG code----------------------//



# Convert the $yum_repo_file names
# to Smart channel names.
reset($yum_repo_file);


while (list($key, $value) = each($yum_repo_file)) {
	$smart_channel_file = str_replace('.repo', '.channel', $yum_repo_file);
}



//------------------DEBUG code----------------------//
if ($DEBUG) {
	reset($smart_channel_file);
	echo "\nContents of \$smart_channel_file[] \n\n";
	while (list($key, $value) = each($smart_channel_file)) {
    	echo "$key: $smart_channel_file[$key] \n";
	}
}
//------------------DEBUG code----------------------//



# Don't create a new Smart Channel file for existing channels.

reset($smart_channel_file);

$no_missing_channels = true;

while (list($key, $value) = each($smart_channel_file)) {
	
	# Is the Smart Channel file missing?
	if (!is_file("$SMART_CHANNELS_DIR/$value")) {
	
		$no_missing_channels = false;
		
		# Build array of missing Smart channel files.
		$missing_smart_channel_file[] = $value;	
		
		# Build array of missing Yum repo files.
    	$missing_yum_repo_file[] = str_replace('.channel', '.repo', $value);
    	
	}
}

if ($no_missing_channels) exit();


//------------------DEBUG code----------------------//
if ($DEBUG) {
	reset($missing_smart_channel_file);

	echo "\nContents of \$missing_smart_channel_file[] \n\n";
	while (list($key, $value) = each($missing_smart_channel_file)) {
    	echo "$key: $missing_smart_channel_file[$key] \n";
	}

	reset($missing_yum_repo_file);

	echo "\nContents of \$missing_yum_repo_file[] \n\n";
	while (list($key, $value) = each($missing_yum_repo_file)) {
    	echo "$key: $missing_yum_repo_file[$key] \n";
	}
	echo "\n";
}
//------------------DEBUG code----------------------//



# Read the *contents* of each missing yum repo file
# into an array.

reset($missing_yum_repo_file);

while (list($key, $value) = each($missing_yum_repo_file)) {
	$yum_repo_list[$value] = file($value);

}



//------------------DEBUG code----------------------//
# echo "\nContents of \$yum_repo_list[] \n\n";
# var_dump($yum_repo_list);
//------------------DEBUG code----------------------//



# Convert the contents of $yum_repo_list[]
# into Smart Package Manager format.

# Array of old text patterns to search for.
$yum_old_pattern = array(
 '$releasever',
 '$basearch',
 "failovermethod=priority",
 "#baseurl=",
 "#mirrorlist=",
 "enabled=1",
 "enabled=0",
 "gpgcheck=1",
 "gpgcheck=0",
 "gpgkey="
);

# New text patterns to replace old text patterns.
$smart_new_pattern = array(
 "$RELEASE_VERSION",
 "$BASE_ARCH",
 "#failovermethod=priority",
 "baseurl=",
 "mirrorlist=",
 "disabled=no",
 "disabled=yes",
 "#gpgcheck=1",
 "#gpgcheck=0",
 "#gpgkey="
);


# Loop over each repo file.
if ($DEBUG) {
	echo "Contents of \$yum_repo_list[] & \$smart_channel_list[] \n";
}
while (list($key, $value) = each($yum_repo_list)) {
    if ($DEBUG) {
	    echo "\n$key: $yum_repo_list[$key] \n";
    }

    # Build the smart channel list.
    $sc_key = str_replace('.repo', '.channel', $key);
    $smart_channel_list[$sc_key] = array();
        
    # Loop over the lines of each yum repo file
    # and convert into smart PM format.
    while (list($line_num, $line_text) = each($yum_repo_list[$key])) {
    	if ($DEBUG) {
    		echo "\nYum repo: $line_num: $line_text";
    	}
		
    	$sc_text = str_replace($yum_old_pattern, $smart_new_pattern, $line_text);

		if ($DEBUG) {
    		echo "Smart PM: $line_num: $sc_text";
		}
		
		# Modify the 'baseurl=' line if it's a Livna repository.
		if (substr_count($key, 'livna')) {
			
			# Skip any URL's of any type.
			if (substr_count($sc_text, "\thttp://", 0)
				|| substr_count($sc_text, "\tftp://", 0)) {
					continue;
				}
			
			# tidy-up the 'baseurl=' parameter.
			$res = substr_count($sc_text, "baseurl=\n", 0);
						
			if (substr_count($sc_text, "baseurl=\n")) {

				$new_baseurl = 'baseurl=';
				
				# Get the next URL in the array,
				# and use that for baseurl=.
				$next_url = $yum_repo_list[$key][$line_num+1];
				$res = str_replace($yum_old_pattern, $smart_new_pattern, $next_url);
				
				$new_baseurl .= str_replace("\t", '', $res);
				
				if ($DEBUG) {
    				echo "\$new_baseurl: $new_baseurl";
				}
				
				$smart_channel_list[$sc_key][$line_num]= $new_baseurl; 
			}
			else {
    			$smart_channel_list[$sc_key][$line_num]= $sc_text; 
			}
		}
        else {
    	    $smart_channel_list[$sc_key][$line_num]= $sc_text;
        } 
    }
}


# Write each Smart channel file out to disk.

$NEW_SMART_CHANNELS_DIR = './new-smart-channels';

# Create the ./new-smart-channels directory
# if it doesn't already exist.

if (!is_dir($NEW_SMART_CHANNELS_DIR)) {
	$res = mkdir($NEW_SMART_CHANNELS_DIR, 0755);
}


reset($smart_channel_list);

if ($DEBUG) {
	echo "Contents of \$smart_channel_list[] \n";
}
while (list($key, $value) = each($smart_channel_list)) {
    if ($DEBUG) {
		echo "\n$key: $smart_channel_list[$key] \n";
    }
    
    # Create & open the new channel file for writing.
    $file_handle = fopen("$NEW_SMART_CHANNELS_DIR/$key", "w");
    
    # Write the Smart channel contents to the new file.
    while (list($line_num, $line_text) = each($smart_channel_list[$key])) {
    	if ($DEBUG) {
    		echo "$line_num: $line_text";
    	}
    	fwrite($file_handle, $line_text);
    	
    	# Output 'type=rpm-md' after the channel name.
    	if (substr_count($line_text, 'name=', 0)) {
    		fwrite($file_handle, "type=rpm-md \n");

    		if ($DEBUG) {
    			echo "type=rpm-md \n";
    		}    		
    	}
    	
    }

    // Close the channel file.
	fclose($file_handle);

}

?>
