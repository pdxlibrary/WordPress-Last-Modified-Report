# WordPress-Last-Modified-Report
This WordPress customization adds a report to the admin dashboard where a list of pages can be viewed and sorted by the date the page was last modified.

## Prep - Advanced Custom Fields and the page_expert field
This report assumes the Primary Contact for a particular page is not necessarily the same person as the page owner. 
To add this data to WordPress for each page, we used the plugin Advanced Custom Fields to add a new custom field for all "page" post type posts. 
This new text field, page_expert, must then be populated for each page on the website by editing each page and setting the new field in the WordPress admin.

## Add Report Section and Pages by Last Modified Date Report to the Admin Dashboard
To add the custom report to the admin dashboard, the following code needs to be added to your theme's functions.php file.

```php

<?php

/* Custom Reports Section */
add_action( 'admin_menu', 'register_reports_menu_page' );
function register_reports_menu_page()
{
	add_menu_page( 'Pages by Last Modified Date', 'Reports', 'manage_options', 'reports_admin', 'reports_admin_page', 'dashicons-analytics', 7 ); 
	add_submenu_page( 'reports_admin', 'Last Modified', 'Last Modified', 'manage_options', 'reports_admin', 'reports_admin_last_modified');
}

function reports_admin_last_modified()
{
	print("<h2>Pages by Last Modified Date</h2>\n");

	$args = array(
		'post_type' => 'page',
		'post_status' => array( 'publish'),
		'posts_per_page' => -1,
		'orderby' => 'modified',
		'order'   => 'ASC',
	);

	$query = new WP_Query($args);

	// The Loop
	if ( $query->have_posts() ) {
?>
	<link rel="stylesheet" type="text/css" href="//cdn.datatables.net/1.10.7/css/jquery.dataTables.css">
	<link rel="stylesheet" type="text/css" href="//cdn.datatables.net/tabletools/2.2.3/css/dataTables.tableTools.min.css">
	<script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
	<script type="text/javascript" charset="utf8" src="//cdn.datatables.net/1.10.7/js/jquery.dataTables.js"></script>
	<script type="text/javascript" charset="utf8" src="//cdn.datatables.net/tabletools/2.2.3/js/dataTables.tableTools.min.js"></script>
	<script>
	$(document).ready( function () {
		$.fn.dataTable.TableTools.defaults.aButtons = [ "copy", "xls", "pdf", "print" ];
		$('#pages_by_modified_date_table').DataTable( {
			"order": [[ 2, "asc" ]],
			"pageLength": 1000,
			dom: 'T<"clear">lfrtip',
			tableTools: {
				"sSwfPath": "//cdn.datatables.net/tabletools/2.2.3/swf/copy_csv_xls_pdf.swf"
			}
		} );
	} );
	</script>

	<table id="pages_by_modified_date_table" border cellpadding="5"><thead><tr><th>Hierarchy</th><th>Page</th><th>Primary Contact</th><th>URL</th><th>Last Modified</th></tr><thead><tbody>
<?php
		while ( $query->have_posts() ) {
			$query->the_post();
			echo '<tr><td>'.implode(" &raquo; ",wordpress_breadcrumbs(get_the_ID())).'</td><td><a href="'.get_the_permalink().'">' . get_the_title() . '</a></td><td>' . get_post_meta( get_the_ID(), 'page_expert', true ) . '</td><td><a href="'.get_the_permalink().'">' . get_the_permalink() . '</a></td><td>' . date("Y-m-d H:i:s",strtotime(get_the_modified_time( "m/d/Y g:i:sa" ))) . '</td></tr>';
		}
		echo '</tbody></table>';
	} else {
		// no posts found
	}
}


function wordpress_breadcrumbs($post_id) {
	
	$breadcrumbs = array();
	
	$post = get_post($post_id);
	
	if ($post->post_parent)
	{
		$parent_id  = $post->post_parent;
		while($parent_id)
		{
			$page = get_page($parent_id);
			$breadcrumbs[] = get_the_title($page->ID);
			$parent_id  = $page->post_parent;
		}
		$breadcrumbs = array_reverse($breadcrumbs);
    }
	else
	{
		if(strcmp($post->post_title,'Home'))
			$breadcrumbs[] = "Home";
	}
	// $breadcrumbs[] = $post->post_title;	// include queried page in breadcrumbs
	return($breadcrumbs);
}

?>

```