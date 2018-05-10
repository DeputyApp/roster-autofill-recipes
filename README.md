# Automagic Customization

Deputy Rostering AutoFill is used for giving further directions/customizations to autofill. Some businesses have some real challenging requirements.

E.g.

* Restaurants: Part timers are cheaper but don’t give too many hours to them!
* Maintenance : Engineers that worked last weekend should not work this weekend.
* Healthcare: Someone should be always be working with First Aid training


# Where

When applying the AutoFill option, you can copy and paste one of the *raw* content of the json files in the advanced section.


# Method

Works by defining a set of statements; statements that have to be true for the entire roster. Each true score gets one point but you can customize exactly how many points if you wish. The higher the score, the better the outcome. You can also give “hints”. Hints are used to force the matching of employees to positions. You are breaking the “genetic search” and forcing specific selections.


# Statement Syntax

{ scorers: [scorer]  }



scorer:
[name:] ,  type:  , params :  , [score : 1] }  # score is optional. Default will be 1. You can set it to 0 or negative (discourage). Give result scoring that can be used with other rules.


Type: field_matches, script, employee_totals, overlap, apply_rest



params

for field_matches          [{ field: ,  data: , type: }]
for script                       scriptname
for employee_totals     {employeePropertyName: , employeePropertyMatch: , total:  ,   type:}
for overlap                    rule_names :  [ rules that it should match]
for apply_rest               api_url:  ,  api_post: , api_column_fetch:  , type: , data:    – make a              deputy_rest call to url, post is a object here. It will get stringified and then __Field Names__ will be searched and replaced








# EXAMPLE

Salary people should work between 40 and 45 hours per week. Also, people may never work during early mornings or late afternoons, regardless of their stress profiles.

```{"scorers":[{"type":"employee_totals","score":10,"params":{"employee_property_name":"Agreement.ContractObject.BasePayRuleObject.RemunerationType","employee_property_match":"2","employee_property_match_type":"eq","compare_value":40,"compare_type":"ge","compare_total":"TotalTime"}},{"type":"employee_totals","score":-20,"params":{"employee_property_name":"Agreement.ContractObject.BasePayRuleObject.RemunerationType","employee_property_match":"2","employee_property_match_type":"eq","compare_value":45,"compare_type":"ge","compare_total":"TotalTime"}},{"type":"field_matches","score":0,"name":"morning_shifts","params":[{"field":"Shift.StartTimeQ.Hour","data":12,"type":"lt"}]},{"type":"field_matches","score":0,"name":"late_shifts","params":[{"field":"Shift.StartTimeQ.Hour","data":18,"type":"ge"}]},{"type":"overlap","score":-10,"params":{"rules":["morning_shifts","late_shifts"],"column":"Shift.Employee"}}]}```



More examples to follow but feel free to contact us.
