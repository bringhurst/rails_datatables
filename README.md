## RailsDatatables

A simpler, Rails-friendly interface to using the DataTables jQuery library from http://datatables.net.

### Prerequisites
Make sure you have jQuery.js and jQuery.dataTables.js in /public/javascripts/ and that they're included in your layout.

### Setup

Give table class of 'datatable'.

Activate using <%= datatable() %>, passing in the columns, how to filter them (sorting type), and any other settings (ajax source, search?, label for search, processing image)

    <% columns = [{:reverse => true, :type => 'html', :class => "first"}, nil, {:class => "last"}] %>
    <%= datatable(@objects.size, columns, {:sort_by => "[0, 'desc']", :processing => image_tag("spinner.gif") }) %>

### Options

#### Table Options

    :sort_by - array, default column number (0 - n-1) and sort order. e.g. "[2, 'desc']". Defaults to initial order.
    :search - boolean, display the search field. Defaults to true.
    :search_label - string, the label for the search field. Defaults to "Search".
    :processing - string, the text or image to display while processing data. Defaults to "Processing".
    :persist_state - boolean, remember the sorting and page of the tables for the user. Defaults to true.
    :additional_data - hash, pass along additional data, such as filter values. Default is none.
    
#### Column Options

    :class - string, the class to assign to the table cell. Default is none.
    :type - string, the type of content in the column, for non-Ajax tables. 'html' will strip all HTML and sort on the inner value, as a string. Default is string.
    :sortable - boolean, allow this column to be sorted on. Default is true.
    :searchable - boolean, allow this column to be searched, for non-Ajax tables. Default is true.

#### AJAX Options
  
  When you're working with large datasets it's not reasonable to load everything on page load. Use an :ajax_source to load just the records that are being displayed, do custom searching (DB, Solr, etc).

    :ajax_source - string, for large datasets, use an ajax source to load each page on its own. For smaller datasets, just load the whole set and let datatable do the sorting

Add a datatable method on your controller to return JSON  
* Return the objects to be displayed  
* Return the total number of objects  
* Add a method to handle sorting - DataTables returns the column that is being sorted (0 - n), so you need to know which column is which and sort on it.
    
### AJAX Example
    
#### Datatable view example - datatable.html.erb

    {"sEcho": <%= params[:sEcho] || -1 %>,
     "iTotalRecords": <%= @total_objects %>,
     "iTotalDisplayRecords": <%= @total_object %>,
     "aaData":[
    <% @objects.each do |object| %>
      ['<%= link_to(object.user.name, user) %>',
       '<%= object.description || "-" %>',
       '<%= object.created_at %>'
      ],
    <% end %>
    ]}

####  Controller example - using will_paginate

    def datatable
      @objects = current_objects(params)
      @total_objectss = total_objects(params)
      render :layout => false
    end

    private
    
    def current_objects(params={})
      current_page = (params[:iDisplayStart].to_i/params[:iDisplayLength].to_i rescue 0)+1
      @current_objects = Object.paginate :page => current_page, 
                                         :include => [:user], 
                                         :order => "#{datatable_columns(params[:iSortCol_0])} #{params[:iSortDir_0] || "DESC"}", 
                                         :conditions => conditions,
                                         :per_page => params[:iDisplayLength]
    end

    def total_objects(params={})
      @total_objects = Object.count :include => [:user], :conditions => conditions
    end

    def datatable_columns(column_id)
      case column_id.to_i
      when 1
        return "objects.description"
      when 2
        return "objects.created_at"
      else
        return "users.name"
      end
    end

    def conditions
      conditions = []
      conditions << "(objects.description ILIKE '%#{params[:sSearch]}%' OR users.name ILIKE '%#{params[:sSearch]}%')" if(params[:sSearch])
      return conditions.join(" AND ")
    end


Copyright (c) 2009 Phronos, released under the MIT license
RailsDatatables
===============

Introduction goes here.


Example
=======

Example goes here.


Copyright (c) 2009 [name of plugin creator], released under the MIT license
