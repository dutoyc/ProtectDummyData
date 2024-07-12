# Protect Interview Task

### Project Brief

Our goal is always to see how customers are behaving, how customers are making bookings and why, how our product is
performing across a range of booking types and values, and ultimately to identify opportunities where we can generate further revenue or commercial value.

### Tools

- Python/ Notepad - Data Preparation
- Python - Data Analysis
- Powerpoint/ PowerBI - Presentation

### Data Cleaning/ Preparation

1. Data loading and observation
2. Checks for missing data
3. Data cleaning and formatting

### Data Analysis

Initial analysis explored the transactional data to answer key questions:

- What is customer behaviour sumarised?
- Which products are performing best?
- Where are there opportunities to most improve revenue generation?

Below is the code used to examine the transaction data using Python:
```Python
import pandas as pd
import matplotlib.pyplot as plt

# Load the data from a CSV file
file_path = '/Users/chrisdutoy/Documents/Work/Interviews/Protect/sample_dyn_for_interview.csv'
data = pd.read_csv(file_path, encoding='iso-8859-1')

# Convert dates to datetime format
data["TransactionDate"] = pd.to_datetime(data["TransactionDate"], format="%d/%m/%Y")
data["EventDate"] = pd.to_datetime(data["EventDate"], format="%d/%m/%Y")

# Adjust the values by dividing by the number of tickets
data["TransactionFaceValueUsdPerTicket"] = data["TransactionFaceValueUsd"] / data["NumberOfTickets"]
data["TransactionConsumerValueUsdPerTicket"] = data["TransactionConsumerValueUsd"] / data["NumberOfTickets"]
data["TransactionProtectionValueUsdPerTicket"] = data["TransactionProtectionValueUsd"] / data["NumberOfTickets"]

# 1. Analyze customer behavior
# Number of opt-ins sold vs. not sold
sold_counts = data["IsSold"].value_counts()
print("Opt-Ins Sold vs. Not Sold:")
print(sold_counts)

# Breakdown of ticket types
ticket_type_counts = data["TicketType"].value_counts()
print("\nTicket Type Counts:")
print(ticket_type_counts)

# Analysis of travel classes
travel_class_counts = data["EventTravelClass"].value_counts()
print("\nTravel Class Counts:")
print(travel_class_counts)

# Analysis of flight types
flight_type_counts = data["FlightType"].value_counts()
print("\nFlight Type Counts:")
print(flight_type_counts)

# 2. Product performance analysis
# Average transaction values per ticket
average_face_value_per_ticket = data["TransactionFaceValueUsdPerTicket"].mean()
average_consumer_value_per_ticket = data["TransactionConsumerValueUsdPerTicket"].mean()
average_protection_value_per_ticket = data["TransactionProtectionValueUsdPerTicket"].mean()

print("\nAverage Transaction Values per Ticket:")
print(f"Face Value: ${average_face_value_per_ticket:.2f}")
print(f"Consumer Value: ${average_consumer_value_per_ticket:.2f}")
print(f"Protection Value: ${average_protection_value_per_ticket:.2f}")

# Performance by ticket type
ticket_type_performance = data.groupby("TicketType").agg({
    "TransactionFaceValueUsdPerTicket": "mean",
    "TransactionConsumerValueUsdPerTicket": "mean",
    "TransactionProtectionValueUsdPerTicket": "mean",
    "IsSold": "sum"
}).reset_index()

print("\nPerformance by Ticket Type:")
print(ticket_type_performance)

# Performance by travel class
travel_class_performance = data.groupby("EventTravelClass").agg({
    "TransactionFaceValueUsdPerTicket": "mean",
    "TransactionConsumerValueUsdPerTicket": "mean",
    "TransactionProtectionValueUsdPerTicket": "mean",
    "IsSold": "sum"
}).reset_index()

print("\nPerformance by Travel Class:")
print(travel_class_performance)

# Performance by travel class and flight type
travel_class_flight_type_performance = data.groupby(["EventTravelClass", "FlightType"]).agg({
    "TransactionFaceValueUsdPerTicket": "mean",
}).reset_index()

# Check the intermediate data
print("\nIntermediate Performance by Travel Class and Flight Type DataFrame:")
print(travel_class_flight_type_performance)

print("\nPerformance by Travel Class and Flight Type:")
print(travel_class_flight_type_performance)

# 3. Identify revenue opportunities
# High and low-performing segments
high_performing_segments = ticket_type_performance.sort_values(by="IsSold", ascending=False)
low_performing_segments = ticket_type_performance.sort_values(by="IsSold")

print("\nHigh Performing Segments:")
print(high_performing_segments.head())

print("\nLow Performing Segments:")
print(low_performing_segments.head())

# Calculate the totals for IsSold, IsClaimed, and IsClaimedSuccess
total_is_sold = data["IsSold"].sum()
total_is_claimed = data["IsClaimed"].sum()
total_is_claimed_success = data["IsClaimedSuccess"].sum()

# Print the results
print(f"Total tickets sold (IsSold): {total_is_sold}")
print(f"Total tickets claimed (IsClaimed): {total_is_claimed}")
print(f"Total successful claims (IsClaimedSuccess): {total_is_claimed_success}")

# Calculate the total number of single and return tickets
total_single_tickets = data[data["FlightType"] == "Single"].shape[0]
total_return_tickets = data[data["FlightType"] == "Return"].shape[0]

# Calculate the number of single and return tickets that have IsSold = 1
sold_single_tickets = data[(data["FlightType"] == "Single") & (data["IsSold"] == 1)].shape[0]
sold_return_tickets = data[(data["FlightType"] == "Return") & (data["IsSold"] == 1)].shape[0]

# Calculate the percentage of single and return tickets that have IsSold = 1
percentage_sold_single_tickets = (sold_single_tickets / total_single_tickets) * 100 if total_single_tickets > 0 else 0
percentage_sold_return_tickets = (sold_return_tickets / total_return_tickets) * 100 if total_return_tickets > 0 else 0

# Print the results
print(f"Percentage of single tickets sold (IsSold = 1): {percentage_sold_single_tickets:.2f}%")
print(f"Percentage of return tickets sold (IsSold = 1): {percentage_sold_return_tickets:.2f}%")

# Calculate the difference in days between TransactionDate and EventDate
data["DaysBetween"] = (data["EventDate"] - data["TransactionDate"]).dt.days

# Group by the number of days between and sum the IsSold, IsClaimed, and IsClaimedSuccess values
sold_claimed_by_days = data.groupby("DaysBetween").agg({
    "IsSold": "sum",
    "IsClaimed": "sum",
    "IsClaimedSuccess": "sum"
}).reset_index()

# Sort by the number of IsSold in descending order and get the top 50
top_50_sold_claimed_by_days = sold_claimed_by_days.sort_values(by="IsSold", ascending=False).head(50)

# Print the result
print("\nTop 50 Days with Highest IsSold Counts:")
print(top_50_sold_claimed_by_days)

# Group by DepartureCountry and sum the IsSold, IsClaimed, and IsClaimedSuccess values
country_stats = data.groupby("DepartureCountry").agg({
    "IsSold": "sum",
    "IsClaimed": "sum",
    "IsClaimedSuccess": "sum"
}).reset_index()

# Sort by the number of IsSold in descending order and get the top 10 countries
top_10_countries = country_stats.sort_values(by="IsSold", ascending=False).head(10)

# Print the result
print("\nTop 10 Countries by IsSold:")
print(top_10_countries)

# Calculate the total number of tickets booked
data["TotalTicketsBooked"] = data["IsSold"] + data["IsClaimed"] + data["IsClaimedSuccess"]

# Calculate the average transaction values per ticket
data["AverageTransactionFaceValuePerTicket"] = data["TransactionFaceValueUsd"] / data["TotalTicketsBooked"]
data["AverageTransactionConsumerValuePerTicket"] = data["TransactionConsumerValueUsd"] / data["TotalTicketsBooked"]
data["AverageTransactionProtectionValuePerTicket"] = data["TransactionProtectionValueUsd"] / data["TotalTicketsBooked"]

# Calculate the average transaction face value per ticket by flight class
avg_face_value_per_flight_class = data.groupby("EventTravelClass")["TransactionFaceValueUsdPerTicket"].mean().reset_index()

# Print the result
print("\nAverage Face Value per Ticket by Flight Class:")
print(avg_face_value_per_flight_class)

# Performance by travel class
travel_class_performance = data.groupby("EventTravelClass").agg({
    "AverageTransactionFaceValuePerTicket": "mean",
    "AverageTransactionConsumerValuePerTicket": "mean",
    "AverageTransactionProtectionValuePerTicket": "mean",
    "TotalTicketsBooked": "sum"
}).reset_index()

print("\nPerformance by Travel Class:")
print(travel_class_performance)

# Calculate the total number of single and return tickets
total_single_tickets = data[data["FlightType"] == "Single"].shape[0]
total_return_tickets = data[data["FlightType"] == "Return"].shape[0]

# Calculate the number of single and return tickets that have IsSold = 1
sold_single_tickets = data[(data["FlightType"] == "Single") & (data["IsSold"] == 1)].shape[0]
sold_return_tickets = data[(data["FlightType"] == "Return") & (data["IsSold"] == 1)].shape[0]

# Calculate the percentage of single and return tickets that have IsSold = 1
percentage_sold_single_tickets = (sold_single_tickets / total_single_tickets) * 100 if total_single_tickets > 0 else 0
percentage_sold_return_tickets = (sold_return_tickets / total_return_tickets) * 100 if total_return_tickets > 0 else 0

# Print the results
print(f"Percentage of single tickets sold (IsSold = 1): {percentage_sold_single_tickets:.2f}%")
print(f"Percentage of return tickets sold (IsSold = 1): {percentage_sold_return_tickets:.2f}%")
```

### Results/ Findings

#### Customer Behaviour:
- 88.8% travelled economy
 - (7.9% first, 2.9% business)
 - Opt-in = 13.4% eco, 16.6% bus, 14.4% first
- Average transaction value per ticket = $1063.21
 -     Business              $3546.58 (R - $3823.76, S - $2604.86)
 -     Economy               $974.30 (R - $1127.04, S - $652.17)
 -     First                 $1069.04 (R - $1234.79, S - $699.95)
*Mark percentage increase from economy - explanation for high volume of firsts - NEEDS A SO WHAT
- 61.2% returns, 33.9% single
  - Opt-in = 14.2% returns, 12.7% single
  - Slight upturn in returns but no significant difference
#### Best Performers/ Revenue Generation Opportunities
- DaysBetween  IsSold
 -         2     305 
 -         3     286         
 -         5     242        
 -         1     229          
 -         6     229         
 -         4     224         
 -         7     211        
 -         8     191       
 -         9     167       
 -         10    158
 - Generally, the closer to the event date, the more frequent the opt-in (>=10 days > 33% of opt-ins)
  - Pricing for opt-in could increase around "last minute" bookings as there is a perceived need for security
-  DaysBetween  IsSold  IsClaimed  IsClaimedSuccess
 -          2     305         16                 5
 -          3     286          8                 1
 -          5     242          7                 3
 -          1     229          6                 0
 -          6     229          7                 1
 -          4     224          4                 1
 -          7     211          4                 0
 -          8     191          3                 1
 -          9     167         11                 4
 -         10     158          5                 1
 -                33.1%       30.6%              25.8%
 -         Top10  2242/6771   71/232             17/66
- Top 10 countries that opt-in:
- Country                      IsSold      IsClaimed          IsClaimedSuccess
 -     Liberia                     803         35                 5
 -     Qatar                       479          7                 3
 -     Djibouti                    476         14                 6
 -     Burundi                     292          6                 4
 -     Tanzania                    277         11                 3
 -     Central African Republic    250          9                 5
 -     South Africa                243          5                 3
 -     Sudan                       199          1                 0
 -     Guinea                      198         11                 3
 -     Niger                       189          2                 0
 -     Top 10 Total                3406       101                32
 -     Total                       6771       232                66
 - Top 10 countries that opt-in account for over half the opt-ins, (nearly half) claims and successful claims
 - Liberia opts-in the most and while claims are high, success % is low making it a good place to target (e.g., make opt-in even more desirable).
 - The others all sit between 1/3 and 1/2, so with opt-in being quite high they could be good places to increase opt-in fees to justify claim success.

### Recommendations

- Spike in first class bookings as it is 3x cheaper than business
 - First being the same price as economy needs some consideration
 - What is the incentive for booking economy?
- Slight upturn in opt-ins for return bookings but not significant enough to justify attention
- Opt-ins are directly correlated to proximity of booking
  - e.g., The closer to the event date, the greater the likelihood of opting in
  - perceive increased risk?/ promotions that want protection
    - Pricing can be adjusted to account for greater activity inside 2-3 weeks of event
- Countries in top 10 list account for over half the total opt-ins
  - Could be cultural/ economic, related to travel quality
  - Nearly half of all claims in top 10 and generally the claims are successful 30-50% of the time
  - Liberia a good country to target as claims are high but success is relatively low

#### Definitions

- Region = This is the region the end customer is from.
- Date received = This is the date that the booking was made by the customer and Protect received via API.
- Date of Departure = This is the date of the customer’s flight departure / start of journey.
- Cancelled? = This is where the customer has cancelled their booking and our client has refunded the customer
themselves (not through us). A scenario might be that a flight has been cancelled, or the customer made a mistake when making a booking and therefore the customer has been refunded the cost of Refund Protect. “1” is “yes”, “0” is “no” and the transaction is active with us. We would recommend to discount these in most part of your analysis and perhaps analyse this separately.
- Opted-in? = This is where the customer has opted-in to our product. “1” is “yes they have opted-in”, and “0” is “no they have not opted-in”.
- Member Fee = This is the fee we invoice our clients (Members) and is a % of each individual Transaction Value in USD.
- Consumer Fee = This is the fee each end-customer is offered our Refund Protect service. Where “opted-in” is “1”, this is the price the customer has purchased our service at. Again, the fee is a % of each individual Transaction Value in USD.
- Member Revenue = This is the ancillary revenue our clients have made on each transaction and is calculated by
subtracting the “Member Fee” from the “Consumer Fee”.
- Transaction Count = This is how many tickets are within a transaction.
- Transaction Value (USD) = This is the total value of booking each customer paid which includes the core booking, our Refund Protect cost, and any extras/services.

