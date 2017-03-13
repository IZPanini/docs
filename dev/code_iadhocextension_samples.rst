==========================
IAdHocExtension Samples
==========================

This article contains integration mode full-code samples for the **IAdHocExtension** APIs, which allow customization code to hook in the report life cycle.

Reference Izenda libraries in a New Project
-------------------------------------------

#. .. _IAdHocExtension_Sample_Project:

   .. figure:: /_static/images/IAdHocExtension_Sample_Project.png
      :align: right
      :width: 706px

      New Project

   Create a new web project in Visual Studio.
#. Name it IAdHocExtensionSample.
#. Select a location (e.g. D:\\Projects).
#. Select to create new solution.
#. Give the solution name IAdHocExtensionSample.
#. Tick the Create directory for solution check-box.
#. Click OK to create the project and solution. |br|
#. Create a new folder for Izenda libraries
   D:\\Projects\\IAdhocExtensionSample\\IAdhocExtensionSample\\izendadll.
#. Copy the file Izenda.BI.Framework.dll from Izenda installation folder
   into the newly-created folder.
#. Open Solution Explorer, right-click References in project
   IAdHocExtensionSample and select Add Reference.
#. In Reference Manager pop-up, click Browse and select the file
   Izenda.BI.Framework.dll (in D:\\Projects\\SetupHiddenFilter).
#. Verify the interface by double-clicking Izenda.BI.Framework in
   References to open Object Browser.
#. Expand the nodes to Izenda.BI.Framework.CustomConfiguration and
   select DefaultAdHocExtension class to see the methods to override.
#. Similarly reference the library Izenda.BI.Core.dll.
#. Also reference System.ComponentModel.Composition (in Assemblies >
   Framework).

.. figure:: /_static/images/DefaultAdHocExtension_class.png

   DefaultAdHocExtension class

Sample Method Implementations
-----------------------------

.. container:: toggle

   .. container:: header

      Full sample code:

   .. code-block:: csharp

      using Izenda.BI.Core;
      using Izenda.BI.Core.QueryEngine;
      using Izenda.BI.Framework.Components.QueryExpressionTree;
      using Izenda.BI.Framework.Components.QueryExpressionTree.Operator;
      using Izenda.BI.Framework.Constants;
      using Izenda.BI.Framework.CustomConfiguration;
      using Izenda.BI.Framework.Models;
      using Izenda.BI.Framework.Models.Common;
      using Izenda.BI.Framework.Models.ReportDesigner;
      using Izenda.BI.Framework.Utility;
      using System;
      using System.Collections.Generic;
      using System.Linq;
      using System.Web;
      using System.ComponentModel.Composition;
      
      namespace IAdHocExtensionSample
      {
         [Export(typeof(IAdHocExtension))]
         public class AdHocExtensionSample : DefaultAdHocExtension
         {
            /// <summary>
            /// Sample Requirement:
            /// If logged user has role Manager --> look up "South America" => "SA", "North America" => "NA"
            /// If logged user has role Employee --> look up "Europe" => "EU"
            /// </summary>
            /// <param name="filterField"></param>
            /// <param name="data"></param>
            /// <returns></returns>
            public override List<string> OnPostLoadFilterData(ReportFilterField filterField, List<string> data)
            {
                 // override dropdown value based on user role for filter on view "OrderDetailsByRegion" and field "CountryRegionName"
                 if (filterField.SourceDataObjectName == "OrderDetailsByRegion" && filterField.SourceFieldName == "CountryRegionName"
                     && (HttpContext.Current.User.IsInRole("Manager") || HttpContext.Current.User.IsInRole("Employee")))
                 {
      
                     // override dropdown's value based on User role
      
                     //Manager, look up "South America" => "SA", "North America" => "NA"
                     if (HttpContext.Current.User.IsInRole("Manager"))
                     {
                         var indexSA = data.IndexOf("South America");
                         if (indexSA != -1)
                             data[indexSA] = "SA";
                         var indexNA = data.IndexOf("North America");
                         if (indexNA != -1)
                             data[indexNA] = "NA";
                     }
      
                     // Employee, look up "Europe" => "EU"
                     if (HttpContext.Current.User.IsInRole("Employee"))
                     {
                         var indexEU = data.IndexOf("Europe");
                         if (indexEU != -1)
                             data[indexEU] = "EU";
                     }
                 }
                 return base.OnPostLoadFilterData(filterField, data);
            }
      
            /// <summary>
            /// Sample Requirement:
            /// If logged user has role Manager --> show some regions ("South America", "North America")
            /// If logged user has role Employee --> show only one region ("Europe")
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override List<ValueTreeNode> OnLoadFilterDataTree(QuerySourceFieldInfo fieldInfo)
            {
                 var result = new List<ValueTreeNode>();
      
                 if (fieldInfo.QuerySourceName == "OrderDetailsByRegion" && fieldInfo.Name == "CountryRegionName"
                     && (HttpContext.Current.User.IsInRole("Manager") || HttpContext.Current.User.IsInRole("Employee")))
                 {
                     //Node [All] is required for UI to render.
                     var rootNode = new ValueTreeNode { Text = "[All]", Value = "[All]" };
                     rootNode.Nodes = new List<ValueTreeNode>();
      
                     if (HttpContext.Current.User.IsInRole("Manager"))
                     {
                         rootNode.Nodes.Add(new ValueTreeNode { Text = "South America", Value = "South America" });
                         rootNode.Nodes.Add(new ValueTreeNode { Text = "North America", Value = "North America" });
                     }
      
                     if (HttpContext.Current.User.IsInRole("Employee"))
                     {
                         rootNode.Nodes.Add(new ValueTreeNode { Text = "Europe", Value = "Europe" });
                     }
      
                     result.Add(rootNode);
                 }
      
                 return result;
            }
      
            /// <summary>
            /// Sample Requirement:
            /// If logged user has role Manager --> show some regions ("South America", "North America")
            /// If logged user has role Employee --> show only one region ("Europe")
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override ReportFilterSetting SetHiddenFilters(SetHiddenFilterParam param)
            {
                 var result = new ReportFilterSetting();
                 var querySource = param.QuerySources.FirstOrDefault(x => x.Name.Equals("OrdersByRegion"));
                 if (querySource == null)
                 {
                     return result;
                 }
      
                 var field = querySource.QuerySourceFields.FirstOrDefault(x => x.Name.Equals("CountryRegionName"));
                 if (querySource != null && field != null)
                 {
                     var equalOperator = Izenda.BI.Framework.Enums.FilterOperator.FilterOperator.EqualsManualEntry.GetUid();
      
                     if (HttpContext.Current.User.IsInRole("Manager"))
                     {
                         // Filter CountryRegionName = South America
                         var reportFilterField1 = new ReportFilterField
                         {
                             QuerySourceId = querySource.Id,
                             SourceDataObjectName = querySource.Name,
                             QuerySourceType = querySource.Type,
                             QuerySourceFieldId = field.Id,
                             SourceFieldName = field.Name,
                             DataType = field.DataType,
                             Position = 1,
                             OperatorId = equalOperator,
                             Value = "South America",
                             RelationshipId = null,
                             IsParameter = false,
                             ReportFieldAlias = null
                         };
      
                         // Filter CountryRegionName = North America
                         var reportFilterField2 = new ReportFilterField
                         {
                             QuerySourceId = querySource.Id,
                             SourceDataObjectName = querySource.Name,
                             QuerySourceType = querySource.Type,
                             QuerySourceFieldId = field.Id,
                             SourceFieldName = field.Name,
                             DataType = field.DataType,
                             Position = 2,
                             OperatorId = equalOperator,
                             Value = "North America",
                             RelationshipId = null,
                             IsParameter = false,
                             ReportFieldAlias = null
                         };
      
                         result.FilterFields = new List<ReportFilterField> { reportFilterField1, reportFilterField2 };
                         result.Logic = "1 OR 2";
                     }
      
                     if (HttpContext.Current.User.IsInRole("Employee"))
                     {
                         // Filter CountryRegionName = Europe
                         var reportFilterField3 = new ReportFilterField
                         {
                             QuerySourceId = querySource.Id,
                             SourceDataObjectName = querySource.Name,
                             QuerySourceType = querySource.Type,
                             QuerySourceFieldId = field.Id,
                             SourceFieldName = field.Name,
                             DataType = field.DataType,
                             Position = 1,
                             OperatorId = equalOperator,
                             Value = "Europe",
                             RelationshipId = null,
                             IsParameter = false,
                             ReportFieldAlias = null
                         };
      
                         result.FilterFields = new List<ReportFilterField> { reportFilterField3 };
      
                     }
                 }
      
                 return result;
            }
      
            /// <summary>
            /// Sample Requirement:
            /// Remove all Map report parts before running
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override ReportDefinition OnPreExecute(ReportDefinition report)
            {
                 if (report.ReportPart.Any(x => x.ReportPartContent.Type == ReportPartContentType.Map))
                 {
                     var filteredReportPart = report.ReportPart.Where(x => x.ReportPartContent.Type != ReportPartContentType.Map).ToList();
                     report.ReportPart = filteredReportPart;
                 }
      
                 return report;
            }
      
            /// <summary>
            /// Sample Requirement:
            /// Limit the execution result to the first 1000 rows only (although the database may return more than that)
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override List<IDictionary<string, object>> OnPostExecute(QueryTree executedQueryTree, List<IDictionary<string, object>> result)
            {
                 return result.Take(1000).ToList();
            }
      
            /// <summary>
            /// Sample Requirement:
            /// Log the queries without result limit operator
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override QueryTree OnExecuting(QueryTree queryTree)
            {
                 var nodeVisitor = new QueryTreePathAnalyzeVisitor(new ExtensibilityFactory(), queryTree.ContextData);
                 nodeVisitor.ContextData = queryTree.ContextData;
                 queryTree.Root.Accept(nodeVisitor);
      
                 var resultLimitOperator = new ResultLimitOperator()
                 {
                     ChildOperand = new Operand()
                     {
                         QuerySource = new QuerySource()
                     }
                 };
      
                 try
                 {
                     nodeVisitor.Visit(resultLimitOperator);
                 }
                 catch (Exception)
                 {
                     Console.WriteLine("LOG: Query with no limit");
                 }
      
                 return queryTree;
            }
      
            /// <summary>
            /// Sample Requirement:
            /// If report filter includes only OrderDetailsByRegion.CountryRegionName, return pre-defined list and skip querying the database
            /// If not, let system query the database
            /// </summary>
            /// <param name="fieldInfo"></param>
            /// <returns></returns>
            public override List<string> OnPreLoadFilterData(ReportFilterSetting filterSetting, out bool handled)
            {
                 handled = false;
                 List<String> result = null;
      
                 if (filterSetting.FilterFields.Count == 1
                     && filterSetting.FilterFields.Any(
                         x   =>  x.SourceDataObjectName.Equals("OrdersByRegion")
                                 && x.SourceFieldName.Equals("CountryRegionName")))
                 {
                     handled = true;
                     result = new List<string>()
                     {
                         "Europe",
                         "North America",
                         "South America"
                     };
                 }
      
                 return result;
            }
         }
      }

Add the New Library
-------------------

#. Build then copy the IAdhocExtensionSample.dll file (it can be found
   at D:\\Projects\\IAdhocExtensionSample\\IAdhocExtensionSample\\bin)
   to the Izenda Back-end API folder (at
   C:\\inetpub\\wwwroot\\Izenda\\API\\bin).
#. Restart the Izenda Back-end website.

UnitTest for the Samples
------------------------

Add UnitTest Project
~~~~~~~~~~~~~~~~~~~~

#. Rick click Solution in Solution Explorer and select Add > New
   Project.
#. Add a Class Library project named AdHocExtensionSampleTest.
#. Reference the project IAdHocExtensionSample (Add Reference and tick
   IAdHocExtensionSample in Projects > Solution).
#. Reference xUnit Library.

   #. Open NuGet Package Manager pop-up from Tools > NuGet Package
      Manager > Manage NuGet Packages for Solution...
   #. Click Browse tab and enter xunit in the text box to search.
   #. Select xunit on the left and tick the AdHocExtensionSampleTest
      project check-box on the right.
   #. Select version 1.9.1 (working at the time of writing) and click
      Install.
   #. Similarly install xunit.runner.visualstudio version 2.1.0 to
      AdHocExtensionSampleTest project.

Implement the UnitTests
~~~~~~~~~~~~~~~~~~~~~~~

#. Right-click the default Class1.cs file in Solution Explorer and rename it to AdHocExtensionSampleTest.cs, also agree to change the class name to AdHocExtensionSampleTest when asked.
#. Implement the tests in xUnit.

.. container:: toggle

   .. container:: header

      Full sample code:

   .. code-block:: csharp

      using System;
      using System.Collections.Generic;
      using Xunit;
      using Izenda.BI.Framework.Models.ReportDesigner;
      using Izenda.BI.Framework.Models;
      using System.Web;
      using System.IO;
      using Izenda.BI.Framework.Constants;
      using Izenda.BI.Framework.Components.QueryExpressionTree;
      
      namespace IAdHocExtensionSample
      {
      
         public class AdhocExtensionSampleTest
         {
            private static Guid querySourceId1 = Guid.Parse("39984D52-C1DE-4388-9ED2-FB7C8C62FD01");
            private static Guid fieldId11 = Guid.Parse("39984D52-C1DE-4388-9ED2-FB7C8C62FD11");
      
            /// <summary>
            /// Test Filter Manager
            /// </summary>
            [Fact]
            public void Execute_HiddenFilter_Manager_Success()
            {
      
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Manager");
      
      
                 var param = new SetHiddenFilterParam()
                 {
                     QuerySources = new List<QuerySource>()
                     {
                         new QuerySource()
                         {
                             Name = "OrdersByRegion",
                             Id = querySourceId1,
                             Type = "Table",
                             QuerySourceFields = new List<QuerySourceField>()
                             {
                                 new QuerySourceField()
                                 {
                                    Name = "CountryRegionName",
                                    Id = fieldId11,
                                    DataType = "varchar"
                                 }
                             }
                         }
                     }
                 };
      
                 var reportFilterSetting = (new AdHocExtensionSample()).SetHiddenFilters(param);
                 Assert.Equal(reportFilterSetting.FilterFields.Count, 2);
                 Assert.Equal(reportFilterSetting.FilterFields[0].Value, "South America");
                 Assert.Equal(reportFilterSetting.FilterFields[1].Value, "North America");
            }
      
            /// <summary>
            /// Test Filter Employee
            /// </summary>
            [Fact]
            public void Execute_HiddenFilter_Employee_Success()
            {
      
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Employee");
      
                 var param = new SetHiddenFilterParam()
                 {
                     QuerySources = new List<QuerySource>()
                     {
                         new QuerySource()
                         {
                             Name = "OrdersByRegion",
                             Id = querySourceId1,
                             Type = "Table",
                             QuerySourceFields = new List<QuerySourceField>()
                             {
                                 new QuerySourceField()
                                 {
                                    Name = "CountryRegionName",
                                    Id = fieldId11,
                                    DataType = "varchar"
                                 }
                             }
                         }
                     }
                 };
      
                 var reportFilterSetting = (new AdHocExtensionSample()).SetHiddenFilters(param);
                 Assert.Equal(reportFilterSetting.FilterFields.Count, 1);
                 Assert.Equal(reportFilterSetting.FilterFields[0].Value, "Europe");
            }
      
            /// <summary>
            /// Test OnLoadFilterDataTree Manager
            /// </summary>
            [Fact]
            public void Execute_OnLoadFilterDataTree_Manager_Success()
            {
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Manager");
      
                 var field = new QuerySourceFieldInfo()
                 {
                     QuerySourceName = "OrderDetailsByRegion",
                     Name = "CountryRegionName"
                 };
      
                 var result = (new AdHocExtensionSample()).OnLoadFilterDataTree(field);
                 Assert.Equal(result.Count,1);
                 Assert.Equal(result[0].Nodes.Count, 2);
                 Assert.Equal(result[0].Nodes[0].Value, "South America");
                 Assert.Equal(result[0].Nodes[1].Value, "North America");
            }
      
            /// <summary>
            /// Test OnLoadFilterDataTree Employee
            /// </summary>
            [Fact]
            public void Execute_OnLoadFilterDataTree_Employee_Success()
            {
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Employee");
      
                 var field = new QuerySourceFieldInfo()
                 {
                     QuerySourceName = "OrderDetailsByRegion",
                     Name = "CountryRegionName"
                 };
      
                 var result = (new AdHocExtensionSample()).OnLoadFilterDataTree(field);
                 Assert.Equal(result.Count, 1);
                 Assert.Equal(result[0].Nodes.Count, 1);
                 Assert.Equal(result[0].Nodes[0].Value, "Europe");
            }
      
            /// <summary>
            /// Test OnPostLoadFilterData Manager
            /// </summary>
            [Fact]
            public void Execute_OnPostLoadFilterData_Manager_Success()
            {
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Manager");
      
                 var field = new ReportFilterField()
                 {
                     SourceDataObjectName = "OrderDetailsByRegion",
                     SourceFieldName = "CountryRegionName"
                 };
      
                 var data = new List<string>()
                 {
                     "Antarctica",
                     "Europe",
                     "North America",
                     "South America"
                 };
      
                 var result = (new AdHocExtensionSample()).OnPostLoadFilterData(field, data);
                 Assert.Equal(result.Count, 4);
                 Assert.NotEqual(result.IndexOf("SA"), -1);
                 Assert.NotEqual(result.IndexOf("NA"), -1);
            }
      
            /// <summary>
            /// Test OnPostLoadFilterData Employee
            /// </summary>
            [Fact]
            public void Execute_OnPostLoadFilterData_Employee_Success()
            {
                 HttpContext.Current = new HttpContext(
                     new HttpRequest("", "http://tempuri.org", ""),
                     new HttpResponse(new StringWriter())
                 );
                 HttpContext.Current.User = new MockUser("Employee");
      
                 var field = new ReportFilterField()
                 {
                     SourceDataObjectName = "OrderDetailsByRegion",
                     SourceFieldName = "CountryRegionName"
                 };
      
                 var data = new List<string>()
                 {
                     "Antarctica",
                     "Europe",
                     "North America",
                     "South America"
                 };
      
                 var result = (new AdHocExtensionSample()).OnPostLoadFilterData(field, data);
                 Assert.Equal(result.Count, 4);
                 Assert.NotEqual(result.IndexOf("EU"), -1);
            }
      
            /// <summary>
            /// Test OnPreExecute
            /// </summary>
            [Fact]
            public void Execute_OnPreExecute_Success()
            {
                 var originalReport = new ReportDefinition()
                 {
                     ReportPart = new List<ReportPartDefinition>
                     {
                         new ReportPartDefinition
                         {
                             ReportPartContent = new ReportPartMap
                             {
                                 Type = ReportPartContentType.Map
                             }
                         },
                         new ReportPartDefinition
                         {
                             ReportPartContent = new ReportPartGrid
                             {
                                 Type = ReportPartContentType.Grid
                             }
                         },
                         new ReportPartDefinition
                         {
                             ReportPartContent = new ReportPartMap
                             {
                                 Type = ReportPartContentType.Map
                             }
                         }
                     }
                 };
      
                 var report = (new AdHocExtensionSample()).OnPreExecute(originalReport);
                 Assert.Equal(report.ReportPart.Count, 1);
            }
      
            /// <summary>
            /// Test OnPostExecute
            /// </summary>
            [Fact]
            public void Execute_OnPostExecute_Success()
            {
                 var originalList = new List<IDictionary<string, object>>();
                 for (var i = 1; i <= 1010; i++)
                 {
                     originalList.Add(new Dictionary<string, object>());
                 }
      
                 var qt = new QueryTree();
      
                 var list = (new AdHocExtensionSample()).OnPostExecute(qt, originalList);
      
                 Assert.Equal(list.Count, 1000);
            }
      
            /// <summary>
            /// Test OnPreLoadFilterData
            /// </summary>
            [Fact]
            public void Execute_OnPreLoadFilterData_Success()
            {
                 var filterSetting = new ReportFilterSetting()
                 {
                     FilterFields = new List<ReportFilterField>()
                     {
                         new ReportFilterField()
                         {
                             SourceDataObjectName = "OrdersByRegion",
                             SourceFieldName = "CountryRegionName"
                         }
                     }
                 };
      
                 bool handled;
      
                 var list = (new AdHocExtensionSample()).OnPreLoadFilterData(filterSetting, out handled);
      
                 Assert.Equal(handled, true);
                 Assert.Equal(list.Count, 3);
            }
         }
      }

Run the UnitTests
~~~~~~~~~~~~~~~~~

#. Open Test Explorer from Menu > Test > Windows.
#. Click Run All in Test Explorer.
#. All the tests should be passed.

.. figure:: /_static/images/IAdhocExtension_TestScripts.png

   Test Result
