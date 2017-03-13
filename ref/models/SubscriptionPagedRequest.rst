

=========================================
SubscriptionPagedRequest
=========================================

.. list-table::
   :header-rows: 1
   :widths: 25 5 60 10

   *  -  Field
      -  NULL
      -  Description
      -  Note
   *  -  **reportId** |br|
         string (GUID)
      -  Y
      -  The id of the report if available
      -
   *  -  **dashboardId** |br|
         string (GUID)
      -  Y
      -  The id of the dashboard if available
      -
   *  -  **isSubscription** |br|
         boolean
      -
      -  Used internally: is this a subscription request
      -
   *  -  **isReportingOwner** |br|
         boolean
      -
      -  Used internally: is current user the report/dashboard owner
      -
   *  -  **createdById** |br|
         string (GUID)
      -  Y
      -  Used internally
      -
   *  -  **subscriptions** |br|
         array of objects
      -
      -  An array of subscriptions, should be populated in case modifications are not yet saved to database
      -
   *  -  **tenantId** |br|
         string (GUID)
      -  Y
      -  The total number of rows
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
