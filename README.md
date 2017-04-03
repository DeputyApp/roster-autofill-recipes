# Automagic Customization

Deputy Rostering AutoFill uses To give further directions/customizations on autofill. Some business have some real crazy stuff that we need to adhere to. E.g.

Resturants: part timers are cheapers but don’t give too many hours to them!
Maintenance : Engineers that worked last weekend, should not work this weekend.
Healthcare: Someone should be always be working with First Aid training


# Where

When applying the AutoFill option, you can copy and paste one of the *raw* content of the json files in the advanced section.


# Method

Works via defining a set of statements.. Statements that all has to be true for the whole roster. Each true score gets a point. you can customize the score. The higher the score, the better the outcome. You can also give “hints”. Hints are used for directly choosing employees at given position. You are breaking the “genetic search” and forcing specific options in there.


# Statement Syntax

{ scorers: [scorer]  }



scorer:
[name:] ,  type:  , params :  , [score : 1] }  # score is optional. default will be 1. You can set it to 0 or negative (discourage). Give result storing that can be used with other rules


Type: field_matches, script, employee_totals, overlap, apply_rest



params

for field_matches          [{ field: ,  data: , type: }]
for script                       scriptname 
for employee_totals     {employeePropertyName: , employeePropertyMatch: , total:  ,   type:}
for overlap                    rule_names :  [ rules that it should match] 
for apply_rest               api_url:  ,  api_post: , api_column_fetch:  , type: , data:    – make a              deputy_rest call to url, post is a object here. It will get stringified and then __Field Names__ will be searched and replaced








# EXAMPLE


## 1) On saturday or sunday shifts don’t schedule full timers!

{ scorers: [{type:field_matches , params:[ 
{field:"Shift.DayOfWeek",data:"Sat",type:"eq”}  , {field:”Shift.EmployeeObject.Agreement. SalaryPayRule”,data:”0”,type:”ne”}]},{type:field_matches , params:[ 
{field:"Shift.DayOfWeek",data:”Sun”,type:"eq”}  , {field:”Shift.EmployeeObject.Agreement. SalaryPayRule”,data:”0”,type:”ne”}]} ]   }


## 2) John and Mary should not work at the same area at the same time

{ scorers: [
{name: ‘john_shifts’ , type:’field_matches’ , params:[ 
{field:"Shift.EmployeeObject.DisplayName”,data:”John%”,type:”lk”}],score:0}, 
{name: ‘mary_shifts’ , type:’field_matches’ , params:[ 
{field:"Shift.EmployeeObject.DisplayName”,data:”Mary%”,type:”lk”}],score:0}, 
{type: overlap , params:[ rules:[john_shifts , mary_shifts] ] , score:-1 } 
]}




## 3) At least one person at all time will should have a first aid certificate training

{ scorers: [
{name: ‘first_aid_shifts’ , type:’field_matches’ , params:[ 
{field:"Shift.EmployeeObject.Training.ModuleObject.ModuleName”,data:”First Aid”,type:”lk”}],score:0}, 
{name: ‘monday_shifts’ , type:’field_matches’ , params:[ {field:"Shift.DayOfWeek",data:”Mon”,type:"eq”}] , score:0} , 
{name: ‘tuesday_shifts’ , type:’field_matches’ , params:[ {field:"Shift.DayOfWeek",data:”Tues”,type:"eq”}] , score:0} , 
{type: overlap , params:[ rules:[john_shifts , monday_shifts] ] , score;1 }, 
{type: overlap , params:[ rules:[first_aid_shifts , tuesday_shifts] ] , score;1 }, 
]}



## 4) Anyone who has employment term PT, give them at least 20 hours, max 22.. But no more

{ scorers: [ 
{type: employee_totals , params:{employeePropertyName: “Agreement. ContractObject.Name” , employeePropertyMatch:”PT” , total: 20  ,   type: ge } , score;2 },
{type: employee_totals , params:{employeePropertyName: “Agreement. ContractObject.Name” , employeePropertyMatch:”PT” , total: 22  ,   type: le } , score;1 } 
]}




## 5) if you worked weekend last week, don’t work weekend this week


{ scorers: [ 
{type: apply_rest , params:{api_url:”resource/Roster/QUERY”  ,  api_post: {"search":{"esearchl":{"field”:”Id”,”type":"eq","data”:”__Shift.Employee__”},”dsearch":{"field”:”Date”,”type":"eq","data”:”__Shift.DateObj.Sub7Day.YMD__”},”satsearch":{"field”:”Date”,”type":"eq","data”:”__objDTWeekStart.Add1Day.Add1Day.Add1Day.Add1Day.Add1Day.YMD__”}}}
 , api_column_fetch: “Id”  , type: “ge” , data:”0”} , score:-1 }
]}



## 6) Do not give someone more than two shifts in one day


{ scorers: [
{name: ‘morning_shifts’ , type:’field_matches’ , params:[ 
{field:"Shift.StartTimeQ.Hour”,data:”12”,type:”lt”}],score:0}, 
{name: ‘afternoon_shifts’ , type:’field_matches’ , params:[ 
{field:"Shift.StartTimeQ.Hour”,data:”12”,type:”ge”}],score:0}, 
{type: overlap , params:[ rules:[morning_shifts , afternoon_shifts] , column : ‘Shift.Employee’ ] , score;-1 }
]}


