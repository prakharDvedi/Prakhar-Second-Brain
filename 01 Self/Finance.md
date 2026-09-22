Create a complete personal finance tracking system in Notion for me.

I want this system to help me:

- log every expense and income transaction
    
- tag transactions properly
    
- track monthly budgets
    
- automatically calculate how much I have spent in each category
    
- show how much budget remains
    
- show my savings rate
    
- track money intentionally allocated toward travel, investing, self-development, and discretionary spending
    
- distinguish actual spending from transfers into savings/investments
    
- review each month
    
- track progress toward larger financial goals
    
- make the system simple enough that I will actually use it consistently
    

Do not create a complicated accounting system.

I want a clean personal finance operating system with interconnected Notion databases, formulas, progress indicators, checkboxes, useful views, and a monthly dashboard.

# PERSONAL FINANCIAL CONTEXT

I am 21 years old.

Current monthly income:

₹150,000/month

This is currently primarily salary/internship income.

I live with my parents.

I currently do not need to pay:

- rent
    
- normal household food expenses
    

Therefore my fixed personal cost of living is unusually low.

My priority right now is aggressive saving, investing in myself, building capital, and eventually creating recurring sources of income outside employment.

My long-term financial objectives include:

- financial independence
    
- creating recurring non-salary income
    
- owning businesses/assets that generate income
    
- building a large investment portfolio
    
- eventually having ₹10 crore+ personal net worth
    
- owning a permanent family home
    
- eventually owning a car and bike
    
- travelling regularly
    
- retaining enough liquidity that major purchases do not destroy my investment portfolio
    
- supporting family when required
    

I currently want approximately ₹1.8 lakh/month in today's money to eventually be covered independently of employment.

I have approximately:

- ₹3 lakh currently in cash/savings
    
- around ₹24,000 currently invested in a Parag Parikh mutual fund
    
- approximately ₹3 lakh of obligations that should eventually be cleared
    
- substantial family real-estate assets, but these should NOT be counted as my normal monthly liquid investment portfolio
    

I have historically contributed around ₹20,000/month toward family expenses, but do NOT make this a mandatory fixed allocation in this template.

Instead create a "Family / Obligations" spending category so I can track whatever I actually contribute each month.

I also want trading/speculation to remain controlled.

Treat trading separately from long-term investing wherever possible.

# CORE MONTHLY ALLOCATION

My baseline monthly income is:

₹150,000

Create these default monthly allocations:

### Self Investment

Budget: ₹10,000/month

Examples:

- gym
    
- fitness
    
- books
    
- technical courses
    
- classes
    
- conferences
    
- certifications
    
- software/tools
    
- career development
    
- networking
    
- education
    

### Guilt-Free Spending

Budget: ₹10,000/month

Examples:

- eating out
    
- clothes
    
- dates
    
- entertainment
    
- social activities
    
- hobbies
    
- discretionary purchases
    

I should be allowed to spend this amount without considering it a financial failure.

### Travel Fund

Allocation: ₹10,000/month

This is primarily a savings bucket rather than an expense bucket.

Unspent money must roll forward over time.

Examples:

- flights
    
- trains
    
- hotels
    
- trips
    
- experiences
    
- travel bookings
    

### Investing + Trading

Allocation: ₹5,000/month minimum

Create two subcategories:

- Long-Term Investing
    
- Trading / Speculation
    

Long-term investing should be treated as asset accumulation, not spending.

Trading should be clearly identifiable separately.

### Core Savings

Everything remaining after the above planned allocations should default to savings.

With ₹150,000 income and these allocations:

Self Investment: ₹10,000  
Guilt-Free: ₹10,000  
Travel: ₹10,000  
Investing/Trading: ₹5,000

Base Core Savings target:

₹115,000/month

This is the default before variable family expenses or unusual expenses.

Do NOT treat this ₹115,000 as an expense.

Treat it as retained capital / savings.

# DATABASE 1: TRANSACTIONS

Create a database called:

## Transactions

Every movement of money should be recorded here.

Create these properties:

### Transaction

Type: Title

Example:  
"Gym membership"  
"Salary"  
"PPFAS SIP"  
"Dinner"  
"Travel Fund Transfer"

### Date

Type: Date

### Amount

Type: Number

Format:  
Indian Rupee ₹

Always store Amount as a positive number.

### Flow Type

Type: Select

Options:

- Income
    
- Expense
    
- Saving
    
- Investment
    
- Trading
    
- Transfer
    

### Category

Type: Select

Options:

- Salary
    
- Self Investment
    
- Guilt-Free
    
- Travel
    
- Long-Term Investment
    
- Trading
    
- Family / Obligations
    
- Health
    
- Transport
    
- Technology
    
- Gifts
    
- Emergency
    
- House
    
- Vehicle
    
- Business
    
- Miscellaneous
    

### Subcategory

Type: Select

Include useful examples:

Self Investment:

- Gym
    
- Course
    
- Book
    
- Software
    
- Conference
    
- Certification
    
- Networking
    
- Equipment
    

Guilt-Free:

- Dining
    
- Clothes
    
- Entertainment
    
- Social
    
- Hobby
    

Travel:

- Flights
    
- Train
    
- Hotel
    
- Food
    
- Local Transport
    
- Activities
    

Investment:

- Mutual Fund
    
- Stock
    
- Gold
    
- Debt
    
- Crypto
    

Trading:

- Stocks
    
- Crypto
    
- Pair Trading
    
- Other
    

Family:

- Household
    
- Sister
    
- Parents
    
- Education
    
- Medical
    
- Other
    

### Account

Type: Select

Options:

- Main Bank
    
- Savings Account
    
- Cash
    
- Investment Account
    
- Trading Account
    
- Credit Card
    
- Other
    

### Need

Type: Select

Options:

- Essential
    
- Useful
    
- Optional
    
- Luxury
    

### Recurring

Type: Checkbox

### Reconciled

Type: Checkbox

Use this after I verify that the transaction matches my bank/account history.

### Notes

Type: Text

### Monthly Bucket

Type: Relation

Relate this to the Monthly Budgets database described below.

# TRANSACTION FORMULAS

Create a formula property called:

## Expense Amount

Formula logic:

If Flow Type is "Expense", return Amount.

Otherwise return 0.

Use the appropriate current Notion formula syntax.

Equivalent logic:

if(  
prop("Flow Type") == "Expense",  
prop("Amount"),  
0  
)

Create:

## Income Amount

Return Amount only when Flow Type = Income.

Create:

## Savings Amount

Return Amount when Flow Type is:

- Saving
    
- Investment
    

Otherwise return 0.

Create:

## Trading Amount

Return Amount when Flow Type = Trading.

Create:

## Month

Automatically output:

YYYY-MM

from the Date property.

For example:

2026-09

Use formatDate().

# DATABASE 2: MONTHLY BUDGETS

Create another database called:

## Monthly Budgets

Each row represents one budget bucket for one specific month.

Properties:

### Budget

Title

Example:

"September 2026 – Self Investment"

### Month

Date

Use the first day of each month.

### Category

Select

Options:

- Self Investment
    
- Guilt-Free
    
- Travel
    
- Investing + Trading
    
- Core Savings
    
- Family / Obligations
    
- Other
    

### Planned Amount

Number formatted ₹

Default monthly values:

Self Investment = ₹10,000  
Guilt-Free = ₹10,000  
Travel = ₹10,000  
Investing + Trading = ₹5,000  
Core Savings = ₹115,000

Family / Obligations = no fixed default.

### Transactions

Relation → Transactions database

### Actual

Rollup

Roll up Amount from related transactions using SUM.

### Remaining

Formula

Planned Amount - Actual

Equivalent logic:

prop("Planned Amount") - prop("Actual")

### % Used

Formula

If Planned Amount is greater than zero:

Actual / Planned Amount

Format as percent.

### Status

Formula

Use these rules:

If Actual = 0:  
"⬜ Not started"

If Actual <= 75% of budget:  
"🟢 Good"

If Actual <= 100%:  
"🟡 Near limit"

If Actual > Planned Amount:  
"🔴 Over budget"

For savings categories reverse the meaning appropriately.

For Core Savings:

If Actual >= Planned Amount:  
"🟢 Target reached"

If Actual >= 80% of Planned Amount:  
"🟡 Almost there"

Otherwise:  
"🔴 Behind target"

# SPECIAL HANDLING FOR TRAVEL

Travel money is a sinking fund.

If I allocate ₹10,000 this month and spend nothing:

the ₹10,000 should remain available next month.

Create a property:

## Travel Fund Balance

If possible, create the necessary rollups/formulas so that:

Previous Travel Balance

- Monthly Travel Contributions
    

- Travel Expenses  
    =  
    Current Travel Fund Balance
    

If automatic cross-month roll-forward is difficult in Notion, create a clean manual "Opening Balance" property.

Formula:

Opening Balance

- Contributions
    

- Expenses  
    =  
    Closing Balance
    

Create both:

Opening Balance  
Closing Balance

# DATABASE 3: MONTHLY FINANCIAL SUMMARY

Create a database:

## Monthly Financial Summary

One row per month.

Example:

September 2026

Properties:

### Month

Date

### Transactions

Relation → Transactions

### Gross Income

Rollup of Income Amount

### Expenses

Rollup of Expense Amount

### Saved + Invested

Rollup of Savings Amount

### Trading Capital

Rollup of Trading Amount

### Net Cash Retained

Formula:

Income - Expenses

Do not subtract investment transfers as consumption.

### Savings Rate

Formula:

Saved + Invested / Gross Income

Format percentage.

Also create:

## Consumption Rate

Expenses / Gross Income

## Free Cash Flow

Income

- actual expenses
    
- family obligations
    

Do not count internal transfers between my own accounts as expenses.

# TARGET SAVINGS RATE

My baseline planned allocation is approximately:

₹115,000 retained/saved from ₹150,000 income before variable family obligations.

Create a target property:

## Target Core Savings

Default:

₹115,000

Create:

## Savings Gap

Actual Core Savings - Target Core Savings

Show positive numbers when ahead and negative when behind.

# MONTHLY DASHBOARD

Create a page called:

# Personal Finance Dashboard

At the top show five large sections/cards:

### 💰 Income

Monthly target:  
₹150,000

Show:  
Actual income this month.

### 🏦 Core Savings

Target:  
₹115,000

Show:  
Actual  
Remaining  
Progress %

### 🧠 Self Investment

Budget:  
₹10,000

Show:  
Spent  
Remaining  
% used

### 🎉 Guilt-Free

Budget:  
₹10,000

Show:  
Spent  
Remaining

### ✈️ Travel Fund

Contribution target:  
₹10,000/month

Show:  
This month's contribution  
Current accumulated balance

### 📈 Investing

Target:  
₹5,000/month minimum

Show separately:

Long-Term Investing  
Trading / Speculation

Do not combine their performance conceptually.

# DASHBOARD VIEWS

Create these linked database views.

## This Month Transactions

Filter:  
Date is within current month.

Sort:  
Newest first.

## Expenses Only

Filter:  
Flow Type = Expense.

## Investments

Filter:

Flow Type = Investment.

## Trading

Filter:

Flow Type = Trading.

## Self Development

Filter:

Category = Self Investment.

## Guilt-Free Spending

Filter:

Category = Guilt-Free.

## Travel

Filter:

Category = Travel.

## Family Support

Filter:

Category = Family / Obligations.

## Needs Review

Filter:

Reconciled is unchecked.

# QUICK ENTRY

Create a very simple transaction entry view.

The minimum required fields should be:

- Transaction
    
- Amount
    
- Category
    
- Date
    

Everything else should be optional.

I should be able to enter something such as:

₹450 dinner

in less than 10 seconds.

# CHECKBOX MONTH-END ROUTINE

Create a section:

## Monthly Money Closeout

Add these checkboxes:

☐ All bank transactions logged

☐ Transactions reconciled

☐ Guilt-Free spending reviewed

☐ Self-investment spending reviewed

☐ ₹10,000 travel contribution made

☐ ₹5,000+ invested

☐ Core savings transferred

☐ Family spending reviewed

☐ Unexpected expenses categorized

☐ Investments updated

☐ Net worth updated

☐ Monthly review written

# MONTHLY REVIEW TEMPLATE

Create this template:

## Income

Salary:  
Other Income:  
Total:

## Spending

Self Investment:  
Guilt-Free:  
Family:  
Health:  
Travel Spending:  
Miscellaneous:

Total Consumption:

## Wealth Building

Core Savings:  
Long-Term Investments:  
Trading Capital:  
Travel Fund Contribution:

Total Capital Retained:

## Ratios

Savings Rate:  
Consumption Rate:

## Reflection

Best financial decision this month:

Worst financial decision:

Largest unnecessary expense:

Best money spent on myself:

Did lifestyle inflation occur?

What should I change next month?

# FINANCIAL GOALS DATABASE

Create one optional database called:

## Financial Goals

Properties:

Goal  
Target Amount  
Current Amount  
Target Date  
Priority  
Status  
Notes

Create initial goals:

### Emergency Reserve

Target:  
₹300,000+

### Clear Existing Obligation

Target:  
₹300,000

### Travel Fund

Ongoing

### Personal Investment Portfolio

Long-term goal

### Employment Independence

Goal:  
eventually generate approximately ₹180,000/month independently of salary in today's purchasing power

### Home Fund

Long-term goal.

The eventual family home may involve both:

- proceeds from existing family real estate
    
- my own accumulated capital
    

Do not assume the entire house must be financed from my salary.

### ₹1 Crore Liquid/Invested Personal Net Worth

### ₹10 Crore+ Personal Net Worth

Long-term objective.

# NET WORTH SECTION

Create a simple database:

## Net Worth

Track monthly balances for:

- Bank Cash
    
- Emergency Fund
    
- Mutual Funds
    
- Stocks
    
- Gold owned personally
    
- Crypto
    
- Trading Capital
    
- Business Equity
    
- Receivables
    
- Personal Real Estate
    
- Other Assets
    

Liabilities:

- Loans
    
- Credit Cards
    
- Personal Obligations
    
- Other Liabilities
    

Formula:

Net Worth =  
Total Assets - Total Liabilities

Keep family/inherited real estate separate from my personal liquid net worth.

Create an optional field:

Family Estate / Future Assets

but DO NOT include this automatically in my Personal Net Worth calculation.

# DESIGN

Make the page visually clean.

Use:

- simple icons
    
- callout boxes
    
- progress bars
    
- compact tables
    
- minimal clutter
    

The top dashboard should answer these questions within 10 seconds:

1. How much did I earn?
    
2. How much did I spend?
    
3. How much did I save?
    
4. Am I within my budgets?
    
5. How much did I invest?
    
6. How much guilt-free money remains?
    
7. How large is my travel fund?
    
8. Is my personal net worth increasing?
    

Do not create decorative complexity.

The system should encourage consistency rather than obsessive expense tracking.

# IMPORTANT ACCOUNTING RULES

Follow these rules throughout the system:

1. Moving ₹50,000 from my salary account to my savings account is NOT ₹50,000 of spending.
    
2. Investing money is wealth allocation, not consumption.
    
3. Depositing money into the Travel Fund is saving.  
    Actually spending it on a trip is an expense.
    
4. Trading capital must remain identifiable separately from long-term investment capital.
    
5. Transfers between my own accounts must not distort income, spending, or savings-rate calculations.
    
6. Salary should only be counted once.
    
7. Family asset values should not inflate my personal liquid net worth.
    
8. Budget performance should be based on actual consumption, not internal transfers.
    
9. Do not punish me for using my ₹10,000 guilt-free spending budget. Its purpose is intentional enjoyment.
    
10. Self-investment spending should remain visible separately because improving my skills, fitness, and earning power is one of my highest priorities.
    

Finally, after creating the system, verify that all relations, rollups, formulas, filters, and views work correctly.

Then create example transactions and confirm that:

- expenses reduce the correct budget
    
- savings do not appear as expenses
    
- investments increase capital retained
    
- transfers do not double-count
    
- travel savings accumulate
    
- the monthly savings rate calculates correctly
    
- budget remaining values update automatically
    

After testing, remove the example transactions so the databases are ready for real use.