# AZ104-Lab 04: Azure Cost Controls

## Business scenario

Northstar Financial Services runs a Pay-as-you-go Azure subscription. Finance wants to know where cloud money goes, be warned before spending drifts, and make sure nothing is billing after experiments finish. As the Azure Infrastructure Engineer, I investigated the subscription's costs, set a budget alarm, reviewed Azure Advisor and proved that no resources were left running.

## Objectives

- Explore costs by resource group, service and tag in Cost Analysis.
- Create a budget with actual-spend alerts.
- Review Azure Advisor recommendations.
- Trace remaining spend and prove nothing is still billing.
- Deliberately test budget validation, then clean up.

## AZ-104 mapping

**Directly tested skills:** monitor and manage cost using Cost Management, budgets and tags; Azure Advisor recommendations.

**Additional engineering skills:** tracing spend to its source, and proving cleanup with the CLI.

## Architecture

No infrastructure was deployed. The lab uses subscription-level Azure Cost Management features only.

```
Subscription
  |-- Cost Analysis (resource group, service, tag views)
  |-- Budget: GBP 10 monthly
  |      |-- Alert at 50% of actual spend --> email
  |      |-- Alert at 90% of actual spend --> email
  |-- Azure Advisor (read-only recommendations)
```

## Azure resources

| Resource | Purpose | Cost |
|---|---|---|
| Budget `northstar-financial-lab` | Spend alarm, GBP 10 monthly, retained | Free |

## Implementation

1. **Cost by resource group and service.** October showed GBP 0.04 of spend from earlier projects.
2. **Cost by tag.** Grouping by tag showed `project: vmss` and untagged spend. The `CostCentre` tag from AZ104-Lab 03 did not appear, because tags only label cost records for billable resources that carried them.
3. **Budget.** Created a GBP 10 monthly budget at subscription scope with two alerts, 50% and 90% of actual spend, sent to my own email. The start date is the first of the month.
4. **Advisor.** The cost category was empty. Two reliability items (soft delete for blobs, NAT gateway) were stale because the resources no longer existed.
5. **Trace the spend.** Remaining costs were storage, virtual network, a small VM charge and bandwidth, all from earlier projects.
6. **Prove it.** The CLI showed no resources and no resource groups.

## Commands

Prove the subscription is empty:

```
az resource list -o table
az group list --query "[].name" -o table
```

Confirm which subscription the CLI is using:

```
az account show --query name -o tsv
```

## Verification evidence

- Budget shows GBP 10, monthly reset, start on the 1st of the month, two actual-spend alerts.
- Budget evaluated spend matched Cost Analysis (GBP 0.04) after it refreshed.
- `az resource list` and `az group list` returned empty output.
- Only one budget exists after the failure test.

## Deliberate failure

I tried to create a second budget with invalid inputs.

- **Start date:** the field is greyed out and fixed to the 1st. Azure removes the invalid choice instead of rejecting it afterwards.
- **Negative amount:** rejected by form validation. A limit below zero cannot trigger a meaningful alert.

I abandoned the form without saving and confirmed no duplicate budget was created.

## Troubleshooting

- **Budget showed GBP 0 against real spend:** budget data refreshes with a delay. It matched Cost Analysis later.
- **Cost Analysis showed an unexpected filter:** clicking a chart segment adds a filter. Remove it with the X on the filter pill.
- **Advisor contradicted the CLI:** Advisor recommended soft delete and a NAT gateway, yet no resources existed. I confirmed the CLI was pointed at my only subscription, concluded Advisor was stale, and trusted the CLI output.

## Cost optimisation

- No billable resources were deployed in this lab.
- Budgets and Advisor are free.
- A budget only warns. It does not stop spending.
- Forecast alerts give earlier warning than actual alerts.
- Tags are not retroactive, and a resource group's tag does not pass to its resources' costs unless tag inheritance is enabled.

## Cleanup

Nothing needed deleting. I verified that no resources or resource groups remain. The budget was deliberately retained as a free safety net.

## Lessons learned

- A budget is a smoke alarm, not a sprinkler.
- Check the real subscription state before trusting a report.
- Spend history and current spend are different things.
- Untagged spend is the problem a required-tag policy prevents.
- Advisor recommends and never acts, and its data can lag.

## Interview talking points

- "I traced a small bill back to deleted earlier projects and confirmed with the CLI that nothing was still billing."
- "I set a monthly budget with actual-spend alerts, and I know when to use forecast alerts instead."
- "I know tags aren't retroactive and don't inherit from resource groups by default."

## Exam quiz

Score: 5/5 (100%).
