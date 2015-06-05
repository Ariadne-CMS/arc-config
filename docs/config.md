ARC: Config
===========

ARC\Config allows you to set values in a tree structure that 'trickle down'. This means that if you set 'color' to 'blue' in a parent node, every child node can get the configured value for 'color' and it will return 'blue'. Unless another node in between has redefined the value.

A concrete code example:

	<?php
		\arc\config::configure('color', 'blue');
		$color = \arc\config::cd('/parent/child/')->acquire('color');
		// => 'blue'

And:

	<?php
		\arc\config::cd('/parent/')->configure('color', 'red');
		$color = \arc\config::cd('/parent/child/')->acquire('color');
		// => 'red'


ARC\Config has the semantics of ARC\Tree, so you can use all the ARC tree methods on it, like: \arc\tree::collapse, \arc\tree::expand, \arc\tree::parents, \arc\tree::dive, etc.

The most usefull are \arc\tree::collapse and ::expand, you could use these to save the tree structure to disk, database, etc. For example:

	<?php
		// write a config file
		$config = \arc\tree::collapse( \arc\config::getConfiguration() );
		$json = json_encode($config);
		file_put_contents('config.json', $json);

	<?php
		// read a config file
		$context = \arc\context::$context;
		$json = file_get_contents('config.json');
		$config = new \arc\config\Configuration( 
			\arc\tree::expand( json_decode($json) )
		)->cd( $context->arcPath );
		\arc\context->push('arcConfig' => $config);

In the code above you should probably read and write the config file not quite like that, but I hope it gets the point across. 

The calls to \arc\context are there to allow you to keep using the static \arc\config calls. \arc\context is something like a dependency injection container. It contains things like 'arcPath' and 'arcConfig', which are used by other ARC factory methods to make the API simple to use. As with all ARC packages, you are not obliged to use the static factory methods, you can create your own objects directly, as indeed I've shown in the code to read a config file:

	<?php
		$config = new \arc\config\Configuration(
			\arc\tree::expand( [
				'/' => ['color' => 'blue'],
				'/parent/' => ['color' => 'red']
			] );
		);

		$color = $config->cd('/parent/child/')->acquire('color'); // => 'red'

The class \arc\config\Configuration has a single dependency in the constructor, it needs an object that functionally implements the \arc\tree\NamedNode class. It may ducktype it, there is no type check.

