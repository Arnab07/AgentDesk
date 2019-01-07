# AgentDesk
Task assignment

Problem statement:

Write the most efficient algorithm that helps s determine a list of matches with match percentages for each match between a huge set of properties (sale and rental) and buyer/renter search criteria as and when a new property or a new search criteria is added to our network by an agent.  

Problem details:
Agentdesks has a lot of properties from property sellers and searches requirements from property buyers which get added to a SQL database every day. Every day these multiple properties and search criteria get added through our application by agents. Write an algorithm to match these properties and search criteria as they come in based on 4 parameters such that each match has a  match percentage.

The 4 parameters are:
•	Distance - radius (high weightage)
•	Budget (high weightage)
•	Number of bedrooms (low weightage)
•	Number of bathrooms (Low weightage)
•	Each match should have a percentage that indicates the quality of the match. Ex: if a property exactly matches a buyers search requirement for all 4 constraints mentioned above, it’s a 100% match.  
•	Each property has these 6 attributes - Id, Latitude, Longitude, Price, Number of bedrooms, Number of bathrooms
•	Each requirement has these 9 attributes - Id, Latitude, Longitude, Min Budget, Max budget, Min Bedrooms required, Max bedroom reqd, Min bathroom reqd, Max bathroom reqd.
Functional requirements
•	 All matches above 40% can only be considered useful.
•	The code should scale up to a million properties and requirements in the system.
•	All corner cases should be considered and assumptions should be mentioned
•	Requirements can be without a min or a max for the budget, bedroom and a bathroom but either min or max would be surely present.
•	For a property and requirement to be considered a valid match, distance should be within 10 miles, the budget is +/- 25%, bedroom and bathroom should be +/- 2.
•	If the distance is within 2 miles, distance contribution for the match percentage is fully 30%
•	If the budget is within min and max budget, budget contribution for the match percentage is full 30%. If min or max is not given, +/- 10% budget is a full 30% match.
•	If bedroom and bathroom fall between min and max, each will contribute full 20%. If min or max is not given, match percentage varies according to the value.
•	The algorithm should be reasonably fast and should be quick in responding with matches for the users once they upload their property or requirement.

# Assumptions:
1.	Latitude(lat) and Longitude(long) are stored in degrees, and search Latitude and Longitude is also in degrees
2.	For property and requirement to be considered a valid match, distance should be within 10 miles, the budget is +/- 25%, bedroom and bathroom should be +/- 2 [All requirements must be met to be even be considered]
3.	Everytime a house or search requirements is added to sql, the following algorithm can be run.
4.	The following algorithm will be used for first time, for next iterations, an additional parameter will be added based on latest addition (if property or search requirement)
  1.	Ex: where p.id=335435, if new property needs to be sorted based on search table
  2.	Ex: where s.id=64164, if new search requirement is added

# Algorithm:
1.	Prepare sql statement (querying made faster using indexing)
Select * from properties p, search s
where and (p.Latitude between s.Latitude -0.145 and Lat+0.145) and (p.Longitude between s.Longitude -0.182 and s.Longitude +0.182) and
(p.price between CalminBudget =CASE
WHEN s.minBudget is NOT NULL THEN s.minBudget
ELSE s.maxBudget*0.5
END
 and CalmaxBudget=CASE
WHEN s.maxBudget is NOT NULL THEN s.maxBudget
ELSE s.minBudget*1.5
END) 
and (p.bedroom between CalminBedroom=CASE
WHEN s.minBedroom is NOT NULL THEN s.minBedroom
ELSE s.maxBedroom+4
END
 and CalmaxBedroom=CASE
WHEN s.maxBedroom is NOT NULL THEN s.maxBedroom
ELSE s.minBedroom-4
END) ) 
and (p.bathroom betweenc minBathroom=CASE
WHEN s. bathroom is NOT NULL THEN s. bathroom
ELSE s. maxBathroom +4
END
 and CalmaxBathroom=CASE
WHEN s.maxBathroom is NOT NULL THEN s.maxBathroom
ELSE s.minBathroom +4
END
)

//Since the sql table is indexed, the query will be faster, and fetch only the probable matches

Note: 
1° degree of latitude = 69miles, therefore 10 miles ≃ 0.145°
1° degree of longitude = 55miles, therefore 10 miles ≃ 0.182°
The (p.lat between Lat-0.145 and Lat+0.145) and (p.long between Long-0.182 and Long+0.182) will search within 10*10 square miles area (next step to make a 10 mile circle)

2.	Loop through the result of the previous step
Foreach row in results
{
	Set row.score =0

	/*execute Haversine formula to find the distance of properties from the desired location
	a = sin²(Δφ/2) + cos φ1 ⋅ cos φ2 ⋅ sin²(Δλ/2)
	c = 2 ⋅ atan2( √a, √(1−a) )
	d = R ⋅ c
Where, φ is latitude, λ is longitude, R is earth’s radius (mean radius = 6,371km≃3959 miles), φ1 = Lat.toRadians(), φ2 =row.lat.toRadians(),  Δφ = (row.lat-Lat).toRadians(), Δλ = (row.lat-Long).toRadians();
NOTE: to execute, longitude and latitude need to be converted to Radian values (toRadians)
	
	/*Score based on distance*/
	If d<=2 miles
		Row.score = row.score + 30		//Requirement 6
	Else if 2<d<=10
		Row.score = row.score + (30-3.75*(d-2))		//(10-2=8 miles=30%, i.e 1 miles=3.75%, i.e 3 miles farther is (30-3.75)% score)
	Else d>10
		Remove row, as it doesn’t fall in 10 miles’ radius, skip the following steps, and reiterate loop
	
	//Score based on budget
	If row.minBudget < row.price < row.maxBudget, if userminBudget and usermaxBudget is set
		Row.score = row.score + 30; 
	Else if row.CalminBudget*1.072 < row.budget < row.CalmaxBudget*1.072, if row.minBudget or  row.maxBudget is not set(+25% from CalminBudget is center value, and +-10 % of centervalue ≃1.072% of (75% of budget))
		Row.score = row.score + 30
	Else
		Row.score=row.score+ (30- |row.price- Budget)|*120/Budget)	
		//center Budget value[Budget] = 1.34*row.CalminBudget
/*between 10% to 25%, it is divided within 30%, hence for any budget, the formula comes as |row.price-Budget|*120/Budget, || will convert every value to positive value
Ex: if user budget is 1000, then min is 0.75*1000=750, assuming the property is 750, and percentage should 0(30-30), since it is farthest, then (1000-750)/1000=>0.25x=30, x=120*/

	//Score based on Bedroom
	If row.minBedroom < row.bedroom < row.maxBedroom, if row.minBedroom and row.maxBedroom is set
		row.score = row.score + 20; 
	Else
		row.score=row.score+ (20- |row.bedroom –row.CalminBedroom+2|*10)	
//between 10% to 25%, it is divided within 30%, hence for any budget, the formula comes as |row.budget-Budget|*120/Budget, || will convert every value to positive value
Ex: if user bedroom req is 4, then min is 2, assuming the property bedroom is 2, and percentage should 0(20-20), since it is farthest, then 4-2=>2x=20, x=10

//Score based on Bathroom, similar to bedroom
	If row.minBathroom < row.bathroom< row.maxBathroom, 	if row.minBathroom and row.maxBathroom is set
		row.score = row.score + 20; 
	Else
		row.score=row.score+ (20- |row.bathroom – row.CalminBathroom+2 |*10)	
	
	if row.score >=40
		array search[req_id][]=row		//append in object
}

3.	Sort the search array based on score/match percentage, and send for viewing
4.	Store the matched data in a noSQL db like Mongodb, hence SQL wouldn’t have to query everytime/everyday, whenever required, details can be taken from mongodb



NOTE: Instead of storing the actual data in SQL, if stored in Mongodb, it has in-built function to find distance. Also very scalable and flexible

