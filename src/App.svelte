<script>
  const taxYearStartDate = 'April 6 2019'
  const taxYearEndDate = 'April 5 2020'

  const daysInTaxYear = 365

  const personalAllowance = 12500

  const incomeTaxBand1Rate = 20
  const incomeTaxBand2Rate = 40
  const incomeTaxBand3Rate = 45

  const incomeTaxBand1Threshold = 0
  const incomeTaxBand2Threshold = 37500
  const incomeTaxBand3Threshold = 150000

  const nic2WeeklyRate = 3

  const nic4Band1Rate = 0
  const nic4Band2Rate = 9
  const nic4Band3Rate = 2

  const nic4Band1Threshold = 0
  const nic4Band2Threshold = 8631
  const nic4Band3Threshold = 50000

  const vatThreshold = 85000

  let daysWorked = 0
  let dayRate = 0
  let allowableExpenses = 0
  let includeToday = true
  let showProjection = true
  let overrideDaysPassed = false
  let overrideDaysPassedValue = null

  $: weeksInTaxYear = daysInTaxYear / 7

  $: daysPassed = showProjection && overrideDaysPassed && overrideDaysPassedValue !== null
    ? overrideDaysPassedValue
    : Math[includeToday ? 'ceil' : 'floor'](
      (Date.now() - new Date(taxYearStartDate)) / 1000 / 60 / 60 / 24
    )

  $: weeksPassed = daysPassed / 7

  $: percentageComplete = daysPassed / (daysInTaxYear / 100)

  $: percentageWorked = daysWorked / (daysPassed / 100)

  $: remainingDays = (vatThreshold - daysWorked * dayRate) / dayRate
  $: remainingDaysPerWeek = Math.min(7, remainingDays / ((daysInTaxYear - daysWorked) / 7))
  $: percentageRemaining = remainingDays / (daysInTaxYear - daysPassed) * 100

  $: projectedAllowableExpenses = allowableExpenses * 100 / percentageComplete

  $: totalIncome = showProjection
    ? (daysInTaxYear * (percentageWorked / 100)) * dayRate
    : daysWorked * dayRate

  $: netIncome = Math.max(
    0,
    totalIncome - (showProjection ? projectedAllowableExpenses : allowableExpenses)
  )

  $: taxableIncome = Math.max(0, netIncome - personalAllowance)

  $: incomeTaxBand1 = Math.min(
    taxableIncome,
    incomeTaxBand2Threshold
  ) * (incomeTaxBand1Rate / 100)

  $: incomeTaxBand2 = Math.min(
    Math.max(0, taxableIncome - incomeTaxBand2Threshold),
    incomeTaxBand3Threshold - incomeTaxBand2Threshold
  ) * (incomeTaxBand2Rate / 100)

  $: incomeTaxBand3 = Math.max(
    taxableIncome - incomeTaxBand3Threshold,
    0
  ) * (incomeTaxBand3Rate / 100)

  $: incomeTax = incomeTaxBand1 + incomeTaxBand2 + incomeTaxBand3

  $: nic2 = showProjection
    ? nic2WeeklyRate * (daysInTaxYear / 7)
    : nic2WeeklyRate * weeksPassed

  $: nic4Band1 = Math.min(
    netIncome,
    nic4Band2Threshold
  ) * (nic4Band1Rate / 100)

  $: nic4Band2 = Math.min(
    Math.max(0, netIncome - nic4Band2Threshold),
    nic4Band3Threshold - nic4Band2Threshold
  ) * (nic4Band2Rate / 100)

  $: nic4Band3 = Math.max(
    netIncome - nic4Band3Threshold,
    0
  ) * (nic4Band3Rate / 100)

  $: nic4 = nic4Band1 + nic4Band2 + nic4Band3

  $: totalTax = incomeTax + nic2 + nic4

  const currency = new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
    minimumFractionDigits: 2,
  })
</script>

<style>
  div {
    display: flex;
    align-items: center;
    margin-top: 16px;
    margin-bottom: 16px;
  }

  div p {
    margin-right: 5px;
  }

  input {
    margin: 0;
  }

  input + label,
  label + input {
    margin-left: 5px;
  }
</style>

<h1>UK Tax Year 2019-2020</h1>
<p>{taxYearStartDate} - {taxYearEndDate}</p>

<hr />

<form>
  <div>
    <p>Show:</p>

    <input
      type="radio"
      bind:group={showProjection}
      value={false}
    />

    <label>Current</label>

    <input
      type="radio"
      bind:group={showProjection}
      value={true}
    />

    <label>Projection</label>
  </div>

  <hr />

  <p>
    Days passed:
    {#if showProjection && overrideDaysPassed}
      <input
        type="number"
        value={overrideDaysPassedValue !== null ? overrideDaysPassedValue : daysPassed}
        on:input={(e) => overrideDaysPassedValue = e.target.value}
        min="0"
      />
    {:else}
      {daysPassed}
    {/if}
  </p>

  <p>Weeks passed: {Math.floor(weeksPassed)}</p>
  <p>Percentage tax year complete: {Math.round(percentageComplete)}%</p>

  <div>
    <input type="checkbox" bind:checked={includeToday} />
    <label>Include today?</label>
  </div>

  {#if showProjection}
    <div>
      <input type="checkbox" bind:checked={overrideDaysPassed} />
      <label>Override days passed?</label>
    </div>
  {/if}

  <hr />

  <div>
    <label>Days worked</label>
    <input type="number" bind:value={daysWorked} min="0" />
  </div>

  <p>Percentage days worked: {Math.round(percentageWorked)}%</p>

  {#if dayRate}
    {#if percentageWorked > percentageRemaining}
      <p>
        <strong>Warning:</strong> Projected to pass the VAT threshold of
        {currency.format(vatThreshold)}. You can work another
        {Math.floor(remainingDays)} days ({Math.floor(percentageRemaining)}% of
        the remaining days. {remainingDaysPerWeek.toFixed(2)} days per week) to
        stay within the VAT threshold.
      </p>
    {:else}
      <p>
        To stay within the VAT threshold you can work another
        {Math.floor(remainingDays)} days ({Math.floor(percentageRemaining)}% of
        the remaining days. {remainingDaysPerWeek.toFixed(2)} days per week).
      </p>
    {/if}
  {/if}

  <div>
    <label>Day rate £</label>
    <input type="number" bind:value={dayRate} step="50" min="0" />
  </div>

  <div>
    <label>Allowable expenses £</label>
    <input type="number" bind:value={allowableExpenses} min="0" />
  </div>
</form>

<hr />

<p>Total income: {currency.format(totalIncome)}</p>
<p>Net income: {currency.format(netIncome)}</p>
<p>Taxable income: {currency.format(taxableIncome)}</p>

<hr />

<p>
  Income tax band 1 ({incomeTaxBand1Rate}%):
  {currency.format(incomeTaxBand1)}
</p>

<p>
  Income tax band 2 ({incomeTaxBand2Rate}%):
  {currency.format(incomeTaxBand2)}
</p>

<p>
  Income tax band 3 ({incomeTaxBand3Rate}%):
  {currency.format(incomeTaxBand3)}
</p>

<p>
  Total income tax: {currency.format(incomeTax)}
</p>

<hr />

<p>
  NIC (class 2): {currency.format(nic2)}
</p>

<hr />

<p>
  NIC (class 4) band 1 ({nic4Band1Rate}%):
  {currency.format(nic4Band1)}
</p>

<p>
  NIC (class 4) band 2 ({nic4Band2Rate}%):
  {currency.format(nic4Band2)}
</p>

<p>
  NIC (class 4) band 3 ({nic4Band3Rate}%):
  {currency.format(nic4Band3)}
</p>

<p>
  Total NIC (class 4): {currency.format(nic4)}
</p>

<hr />

<p>
  Total tax:
  {currency.format(totalTax)}
  {showProjection ? `(${currency.format(totalTax / 12)} per month)` : ``}
</p>