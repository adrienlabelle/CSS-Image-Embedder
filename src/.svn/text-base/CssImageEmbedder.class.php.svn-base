<?php 
 /* 
  * Copyright (c) 2011 Adrien Labelle <adrien.labelle@gmail.com>
  * 
  * Permission is hereby granted, free of charge, to any person obtaining a copy
  * of this software and associated documentation files (the "Software"), to deal
  * in the Software without restriction, including without limitation the rights
  * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
  * copies of the Software, and to permit persons to whom the Software is
  * furnished to do so, subject to the following conditions:
  * 
  * The above copyright notice and this permission notice shall be included in
  * all copies or substantial portions of the Software.
  * 
  * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
  * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
  * THE SOFTWARE.
  * 
  */

/**
 * Generate CSS files embedding images called fron style
 * 
 */
class CssImageEmbedder {
	
	/**
	 *  properties
	 */
	protected $_rootUrl = NULL;
	protected $_cssPath = NULL;
	protected $_destination = '.';
	protected $_currentStylesheet = NULL;
	protected $_sources = array();
	protected $_separator = NULL;
	protected $_types = array();
	protected $_mhtmlLocationIndex = 0;
	
	/**
	 * constructor
	 */
	public function __construct() {
		$this->_types = array(
			'.jpg'	=> 'image/jpg',
			'.jpeg'	=> 'image/jpg',
			'.png'	=> 'image/png',
			'.gif'	=> 'image/gif'
		);
	}
	
	/**
	 * set the root URL - the URL where the CSS files are accessible
	 * 
	 */
	public function setRootUrl($url) {
		$this->_rootUrl = $url;
		print "[info]	Root URL set to $url\n";
	}
	
	/**
	* set the CSS path - the filesystem path where the CSS files are
	*
	*/
	public function setCssPath($path) {
		if(!is_dir($path)) {
			
			print "[error]	$path doesn't exist, it cannot be used as the CSS path.\n";
		
		} else if(!is_readable($path)) {
			print "[error]	$path is not readable, it cannot be used as the CSS path.\n";
			
		} else {
			$this->_cssPath = $path;
			print "[info]	CSS path set to $path\n";
		}
	}
	
	/**
	* set the destination path - where the CSS files will be generated
	*
	*/
	public function setDestination($path) {
		
		if(!is_dir($path)) {
				
			print "[error]	$path doesn't exist, it cannot be used as the destination path.\n";
		
		} else if(!is_writeable($path)) {
			print "[error]	$path is not writeable, it cannot be used as the destination path.\n";
					
		} else {
			$this->_destination = $path;
			print "[info]	destination path set to $path\n";
		}
	}
	
	/**
	 * Add a CSS file for processing
	 */
	public function add($path) {
		$location = $this->_cssPath.$path;
		
		if(is_null($this->_cssPath)) {
			print "[error] please set CSS path before adding stylesheets\n";
		}
		
		if(file_exists($location) && is_readable($location)) {
			$this->_sources[$path] = file_get_contents($this->_cssPath.$path);
			print '[info]  stylesheet successfully added: '.$path."\n";
			
		} else {
			print '[error] cannot find or read stylesheet: '.$location."\n";
		}
	}
	
	/**
	 * Generate CSS files with embed images
	 * 
	 */
	public function run() {
		foreach($this->_sources as $path => $content) {
			
			/* W3C-Compliant browsers */
			print "[info]	generating ".$path." (data URLs)\n";
			$this->_outputStylesheet($path, $content, 'w3c');
			
			/* IE */
			print "[info]	generating ".$path." (MHTML)\n";
			$this->_outputStylesheet($path, $content, 'ie');
		}
	}
	
	/**
	 * Generate a single CSS file according to the content and the specs provided
	 * 
	 */
	protected function _outputStylesheet($path, $content, $specs="w3c", $destination='.') {
		
		/* set the current CSS file path to be retrieved by other methods */
		$this->_currentStylesheet = $this->_rootUrl.$path;
		
		/* Internet Explorer -  MHTML */
		if($specs == 'ie') {
			
			/* add MHTML header */
			$this->_currentMhtml = "/*\n";
			$this->_currentMhtml .= "Content-Type: multipart/related; boundary=\"".$this->_getSeparator(false)."\"\n";
			$this->_currentMhtml .= "\n";
		}
		
		/* parse and replace the url(...) values */
		$content = $this->_parseAndReplace($content, $specs);
		
		/* Internet Explorer -  MHTML */
		if($specs == 'ie') {
			
			/* add MHTML end separator */
			$this->_currentMhtml .= $this->_getSeparator(true, true)."\n";
			$this->_currentMhtml .= "*/\n\n";
			file_put_contents($destination.'/ie-'.$path, $this->_currentMhtml.$content);
			
			/* reset global value */
			$this->_currentMhtml = NULL;
		
		/* W3C-compliant browsers -  data URL, base64-encoded */
		} else {
			file_put_contents($destination.'/'.$path, $content);
		}
		
		/* reset global value */
		$this->_currentStylesheet = NULL;
	}
	
	/**
	 *  parse the CSS file line by line and replace the url(...) values by the relevant value (either data URL or MHTML location)
	 *  
	 */
	protected function _parseAndReplace($content, $specs) {
		
		/* split file content into lines */
		$lines = explode("\n", $content);
		
		/* loop through lines */
		foreach($lines as $idx => $line) {
			
			/* convert any url(...) values in the line */
			$convertMethod = ($specs == 'ie')?'_convertIe':'_convert';
			$line = preg_replace_callback('@url\((.*?)\)@si', array(&$this, $convertMethod), $line);
			
			if($line) {
				$lines[$idx] = $line;
			}
		}
		
		/* put the lines together again. */
		return implode("\n", $lines);
	}
	
	/**
	 * conversion function for W3C-compliant browsers (data URLs)
	 * 
	 */
	protected function _convert($matches) {
		$resource = $matches[1];
		$resource = trim($resource, '\'"');
		$absResource = $this->_absoluteURL($resource, $this->_rootUrl);
		$base64Data = $this->_getBase64Data($absResource, $this->_currentStylesheet);
		
		if($base64Data) {
			return 'url("'.$base64Data['dataUri'].'")';
		
		} else {
			return $matches[0];
		}
	}
	
	/**
	 * conversion function for Internet Explorer (MHTML)
	 *
	 */
	protected function _convertIe($matches) {
		$resource = $matches[1];
		$resource = trim($resource, '\'"');
		$absResource = $this->_absoluteURL($resource, $this->_rootUrl);
		$base64Data = $this->_getBase64Data($absResource, $this->_currentStylesheet);
		
		if($base64Data) {
			$this->_currentMhtml .= $base64Data['mhtml']['part'];
			return 'url("'.$base64Data['mhtml']['url'].'")';
		
		} else {
			return $matches[0];
		}
	}
	
	/**
	 * Provide a MHTML separator (multipart boundary)
	 * 
	 */
	protected function _getSeparator($prefix=true, $suffix=false) {
		if(is_null($this->_separator)) {
			$this->_separator = hash('md5', 'mhtml_boundary_'.time());
		}
		return ($prefix?'--':'').$this->_separator.($suffix?'--':'');
	}
	
	/**
	* Provide a MHTML location identifier
	*
	*/
	protected function _getIdentifier() {
		$this->_mhtmlLocationIndex++;
		return $this->_indexToChars($this->_mhtmlLocationIndex);
	}
	
	/**
	* Provide a MHTML location identifier ()
	*
	*/
	protected function _getBase64Data($imagePath, $stylesheetUrl) {
		$extension = strrchr($imagePath, '.');
		
		if(!in_array($extension, array_keys($this->_types))) {
			print "[error]  image type not supported: ".$imagePath."\n";
			return false;
		}
		$type = $this->_types[$extension];
		$imgContent = @file_get_contents($imagePath);
		
		if($imgContent === FALSE) {
			print "[error]	Could not retrieve image: ".$imagePath."\n";
			return false;
		}
		$base64 = base64_encode($imgContent);
		

		/* generate data URI */
		$dataUri = 'data:'.$type.';base64,'.$base64;
		
		/* generate MHTML part */
		$identifier = $this->_getIdentifier();
		$mhtmlPart = $this->_getSeparator()."\n";
		$mhtmlPart .= 'Content-Location:'.$identifier."\n";
		$mhtmlPart .= "Content-Transfer-Encoding:base64\n";
		$mhtmlPart .= "\n";
		$mhtmlPart .= $base64."\n";
		
		/* generate MHTML URL */
		$mhtmlUrl = 'mhtml:'.$stylesheetUrl.'!'.$identifier;
		
		return array(
			'dataUri'	=> $dataUri,
			'mhtml'		=> array(
				'part'	=> $mhtmlPart,
				'url'	=> $mhtmlUrl
			)
		);
	}
	
	/**
	 * provide a n letter identifier calculated with an index value
	 * 
	 */
	protected function _indexToChars($index, $start=66, $end=90) {
	    $sig = ($index < 0);
	    $index = abs($index);
	    $str = "";
	    $cache = ($end-$start);
	    while($index != 0) {
	        $str = chr(($index%$cache)+$start-1).$str;
	        $index = ($index-($index%$cache))/$cache;
	    }
	    if($sig) {
	        $str = "-".$str;
	    }
	    return $str;
	}
	
	/**
	* convert a relative URL to an absolute URL
	*
	*/
	protected function _absoluteUrl($rel, $base) {
        /* return if already absolute URL */
        if (parse_url($rel, PHP_URL_SCHEME) != '') {
        	return $rel;
        }

        /* queries and anchors */
        if ($rel[0]=='#' || $rel[0]=='?') {
        	return $base.$rel;
        }

        /* parse base URL and convert to local variables: $scheme, $host, $path */
        extract(parse_url($base));

        /* remove non-directory element from path */
        $path = preg_replace('#/[^/]*$#', '', $path);

        /* destroy path if relative url points to root */
        if ($rel[0] == '/') {
        	$path = '';
        }

        /* dirty absolute URL */
        $abs = "$host$path/$rel";

        /* replace '//' or '/./' or '/foo/../' with '/' */
        $re = array('#(/\.?/)#', '#/(?!\.\.)[^/]+/\.\./#');
        for($n=1; $n>0; $abs=preg_replace($re, '/', $abs, -1, $n)) {}
		
        /*  if '../' is left, it cannot be resolved, remove it */
        $abs = str_replace('/../', '/', $abs);
        
        /* return absolute URL */
        return $scheme.'://'.$abs;
    }
	
}
?>