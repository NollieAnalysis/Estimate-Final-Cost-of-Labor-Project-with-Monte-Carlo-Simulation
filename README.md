# Estimate-Final-Cost-of-Labor-Project-with-Monte-Carlo-Simulation

# Project Objectives:
First we gather an initial estimate of days and daily cost for project tasks based on hourly wage rate, number of workers, quantity of work to perform, labor production rate, and working hours per day.

After the initial estimate we use those parameters, along with a standard deviation rate (for uncertainty level) and specified simulation count, to model different working scenarios that result in different production rates. The varying production rates will directly effect final task costs and overall project cost.

This simulation is relevant due to the difficulty of achieving consistent production rates on a construction project. Any factor of items can effect daily rates. Examples include, other trades interfering with your work area, not enough material, bad weather, and material not close enough to work area. The resulting statistics can be helpful in budgeting for various project costs. Furthermore, using the outputted mean and standard deviation can provide confidence and structure to project estimates and or budgeting exercises.

# Project Deliverables:
# Python Code

```python

'''
monte-carlo-simulation-example-code-cost-completing-project-2-labor-and-productions(REV_7)(worker-count-fixed)(working_code)

In REV_6, normal distribution changed to log normal distribution in order to have ability to increase 'std_dev' (for more uncertainty) and avoid negative cost (for example, negative minimum cost when 'std_dev' is high)
'''

# packages
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Task details with a fixed number of workers
tasks = {
    'Task 1': {'base_cost': 0, 'hourly_wage_rate': 100, 'num_workers': 3, 'quantity': 500, 'prod_rate_day': 50, 'std_dev': 0.1},
    'Task 2': {'base_cost': 0, 'hourly_wage_rate': 100, 'num_workers': 5, 'quantity': 1000, 'prod_rate_day': 25, 'std_dev': 0.1},
    'Task 3': {'base_cost': 0, 'hourly_wage_rate': 100, 'num_workers': 4, 'quantity': 250, 'prod_rate_day': 30, 'std_dev': 0.1}
}

# Number of working hours per day
hours_per_day = 8

# Initial estimates
initial_estimates = {}
initial_estimates2 = {}
for task, details in tasks.items():
    # Estimate duration with the fixed number of workers
    estimated_duration = details['quantity'] / (details['num_workers'] * details['prod_rate_day'])
    initial_estimates[task] = estimated_duration

    # Estimate daily rate cost
    daily_rate_cost = details['num_workers'] * details['hourly_wage_rate'] * hours_per_day
    initial_estimates2[task] = daily_rate_cost

print("Initial Estimates (in days):")
for task, duration in initial_estimates.items():
    print(f"{task}: {duration:.2f} days")

print("Initial estimate for daily rate cost:")
for task, cost in initial_estimates2.items():
    print(f"{task}: {cost:.2f} dollars")

# Monte Carlo simulation
num_simulations = 50000
total_costs = []
task_costs_all = {task: [] for task in tasks.keys()}
task_durations_all = []

for _ in range(num_simulations):
    total_cost = 0
    task_durations = {}
    task_costs = {}

    for task, details in tasks.items():
        # Use the fixed number of workers for each task
        num_workers = details['num_workers']

        # Generate a production rate based on the log-normal distribution
        mu = np.log(details['prod_rate_day'])
        sigma = details['std_dev']
        production_rate = np.random.lognormal(mu, sigma)

        # Calculate task duration in days
        task_duration = details['quantity'] / (num_workers * production_rate)
        task_durations[task] = task_duration

        # Calculate task cost based on hourly wage rate, number of workers, and task duration
        task_cost = details['base_cost'] + (task_duration * num_workers * details['hourly_wage_rate'] * hours_per_day)
        task_costs[task] = task_cost

        # Update total cost
        total_cost += task_cost

    total_costs.append(total_cost)
    task_durations_all.append(task_durations)
    for task in task_costs:
        task_costs_all[task].append(task_costs[task])

# Convert results to DataFrames
results_df = pd.DataFrame(total_costs, columns=['Total_Cost'])
durations_df = pd.DataFrame(task_durations_all)
costs_df = pd.DataFrame(task_costs_all)

# Print summary statistics for total costs
print(f"Mean of total costs: {results_df['Total_Cost'].mean():.2f}")
print(f"Standard deviation of total costs: {results_df['Total_Cost'].std():.2f}")
print(f"5th percentile: {np.percentile(total_costs, 5):.2f}")
print(f"95th percentile: {np.percentile(total_costs, 95):.2f}")
print("Summary statistics for total cost:\n", results_df['Total_Cost'].describe().apply(lambda x: f'{x:.2f}'))

# Summary statistics for each task cost
print("\nSummary statistics for each task cost:")
print(costs_df.describe().applymap(lambda x: f'{x:.2f}'))

# Variance in initial estimates and simulated durations for each task
variance_data = []
for task, duration in initial_estimates.items():
    simulated_duration = durations_df[task].mean()
    difference = simulated_duration - duration
    variance_data.append({
        'Task': task,
        'Initial Estimate': duration,
        'Simulated Duration': simulated_duration,
        'Difference': difference
    })

variance_df = pd.DataFrame(variance_data)

# Display variance table
print("\nVariance in Initial Estimates and Simulated Durations:")
print(variance_df)




    # plots

# Plot the distribution of total costs
plt.figure(figsize=(10, 6))
plt.hist(results_df['Total_Cost'], bins=50, alpha=0.75)
plt.title('Distribution of Total Project Costs')
plt.xlabel('Total Cost')
plt.ylabel('Frequency')
plt.show()

# Plot the distribution of each task cost
plt.figure(figsize=(10, 6))
for task in tasks.keys():
    plt.hist(costs_df[task], bins=50, alpha=0.75, label=task)
plt.title('Distribution of Task Costs')
plt.xlabel('Task Cost')
plt.ylabel('Frequency')
plt.legend()
plt.show()

# Plot the difference between initial estimates and simulated durations
plt.figure(figsize=(10, 6))
plt.bar(variance_df['Task'], variance_df['Difference'], color='skyblue')
plt.title('Difference Between Initial Estimates and Simulated Durations')
plt.xlabel('Task')
plt.ylabel('Difference (days)')
plt.xticks(rotation=45)
plt.grid(axis='y')
plt.show()


'''
The individual task costs are being summarized separately from the total project cost. The summary statistics for the individual task costs are based on the costs for each task, while the summary statistics for the total cost are based on the combined costs of all tasks for each simulation.

While the mean of the total costs equals the sum of the means of the individual task costs (because expectation is linear), the other summary statistics (such as standard deviation, minimum, maximum, percentiles) do not follow this additive property. The distribution of the total cost is affected by the combined variability and correlations between the individual task costs, which leads to different summary statistics.

Here's a clearer explanation:

    Mean: The mean of the total cost is the sum of the means of the individual task costs, which is why they match.
    
    Standard Deviation: The standard deviation of the total cost is not simply the sum of the individual standard deviations. The total standard deviation is influenced by the individual task standard deviations and any correlations between tasks.
    
    Minimum/Maximum: The minimum and maximum of the total cost are derived from the combined distribution, which is not directly related to the minimum and maximum of individual task costs.
    
    Percentiles: Percentiles (25th, 50th, 75th) are also not additive. They depend on the overall distribution of total costs, which is shaped by how individual task costs vary and combine in each simulation.
'''

```
