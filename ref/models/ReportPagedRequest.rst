

=========================================
ReportPagedRequest
=========================================

.. list-table::
   :header-rows: 1
   :widths: 25 5 60 10

   *  -  Field
      -  NULL
      -  Description
      -  Note
   *  -  **categoryId** |br|
         string (GUID)
      -  Y
      -  The id of the category if available
      -
   *  -  **subCategoryId** |br|
         string (GUID)
      -  Y
      -  The id of the sub-category if available
      -
   *  -  **isGlobal** |br|
         boolean
      -  Y
      -  Whether to load global reports
      -
   *  -  **timestampOffset** |br|
         real number
      -
      -  The timestamp offset. See :doc:`/ui/doc_user_setup`.
      -
   *  -  **tenantId** |br|
         string (GUID)
      -  Y
      -  The id of the tenant 
      -  Inherited from :doc:`PagedRequest`
   *  -  **criteria** |br|
         array of objects
      -
      -  An array of :doc:`SearchCriteria` objects
      -  Inherited from :doc:`PagedRequest`
   *  -  **sortOrders** |br|
         array of objects
      -
      -  An array of :doc:`SortOrder` objects
      -  Inherited from :doc:`PagedRequest`
   *  -  **pageIndex** |br|
         integer
      -
      -  The index of the page
      -  Inherited from :doc:`PagedRequest`
   *  -  **pageSize** |br|
         integer
      -
      -  The size of the page
      -  Inherited from :doc:`PagedRequest`
   *  -  **total** |br|
         integer
      -
      -  The total number of rows
      -  Inherited from :doc:`PagedRequest`
   *  -  **skipItems** |br|
         integer
      -
      -  Skip items
      -  Inherited from :doc:`PagedRequest`
   *  -  **isLastPage** |br|
         boolean
      -
      -  Whether this is the last page
      -  Inherited from :doc:`PagedRequest`

.. container:: toggle

   .. container:: header

      **Sample**:

   .. code-block:: json

      {
        	"subcategoryid" : null,
        	"categoryId" : null,
        	"tenantId" : null,
        	"pageSize" : 10,
        	"pageIndex" : 1,
        	"sortOrders" : [{
        			"key" : "reportname",
        			"descending" : true
        		}
        	],
        	"criteria" : [{
        			"key" : "reportName",
        			"value" : "test",
        			"operation" : 1
        		}
        	]
      }
