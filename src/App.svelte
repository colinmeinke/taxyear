<script>
  const taxYears = [
    {
      taxYearStartDate: 'April 6 2018',
      taxYearEndDate: 'April 5 2019',
      taxYearEnded: Date.now() > new Date('April 5 2019'),
      daysInTaxYear: 365,
      personalAllowance: 11850,
      incomeTaxBand1Rate: 20,
      incomeTaxBand2Rate: 40,
      incomeTaxBand3Rate: 45,
      incomeTaxBand1Threshold: 0,
      incomeTaxBand2Threshold: 34500,
      incomeTaxBand3Threshold: 150000,
      nic2WeeklyRate: 2.95,
      nic4Band1Rate: 0,
      nic4Band2Rate: 9,
      nic4Band3Rate: 2,
      nic4Band1Threshold: 0,
      nic4Band2Threshold: 8424,
      nic4Band3Threshold: 46350,
      vatThreshold: 85000,
    },
    {
      taxYearStartDate: 'April 6 2019',
      taxYearEndDate: 'April 5 2020',
      taxYearEnded: Date.now() > new Date('April 5 2020'),
      daysInTaxYear: 366,
      personalAllowance: 12500,
      incomeTaxBand1Rate: 20,
      incomeTaxBand2Rate: 40,
      incomeTaxBand3Rate: 45,
      incomeTaxBand1Threshold: 0,
      incomeTaxBand2Threshold: 37500,
      incomeTaxBand3Threshold: 150000,
      nic2WeeklyRate: 3,
      nic4Band1Rate: 0,
      nic4Band2Rate: 9,
      nic4Band3Rate: 2,
      nic4Band1Threshold: 0,
      nic4Band2Threshold: 8632,
      nic4Band3Threshold: 50000,
      vatThreshold: 85000,
    },
    {
      taxYearStartDate: 'April 6 2020',
      taxYearEndDate: 'April 5 2021',
      taxYearEnded: Date.now() > new Date('April 5 2021'),
      daysInTaxYear: 365,
      personalAllowance: 12500,
      incomeTaxBand1Rate: 20,
      incomeTaxBand2Rate: 40,
      incomeTaxBand3Rate: 45,
      incomeTaxBand1Threshold: 0,
      incomeTaxBand2Threshold: 50000,
      incomeTaxBand3Threshold: 150000,
      nic2WeeklyRate: 3.05,
      nic4Band1Rate: 0,
      nic4Band2Rate: 9,
      nic4Band3Rate: 2,
      nic4Band1Threshold: 0,
      nic4Band2Threshold: 9500,
      nic4Band3Threshold: 50000,
      vatThreshold: 85000,
    },
  ]

  let selectedTaxYear = taxYears.length - 1
  let daysWorked = 0
  let dayRate = 0
  let allowableExpenses = 0
  let includeToday = true
  let showProjection = false
  let overrideDaysPassed = false
  let overrideDaysPassedValue = null

  $: taxYear = taxYears[selectedTaxYear]

  $: daysPassed = taxYear.taxYearEnded
    ? taxYear.daysInTaxYear
    : (showProjection && overrideDaysPassed && overrideDaysPassedValue !== null
      ? overrideDaysPassedValue
      : Math[includeToday ? 'ceil' : 'floor'](
        (Date.now() - new Date(taxYear.taxYearStartDate)) / 1000 / 60 / 60 / 24
      )
    )

  $: weeksPassed = Math.ceil(daysPassed / 7)

  $: percentageComplete = daysPassed / (taxYear.daysInTaxYear / 100)

  $: percentageWorked = daysWorked / (daysPassed / 100)

  $: remainingDays = (taxYear.vatThreshold - daysWorked * dayRate) / dayRate
  $: remainingDaysPerWeek = Math.min(7, remainingDays / ((taxYear.daysInTaxYear - daysWorked) / 7))
  $: percentageRemaining = remainingDays / (taxYear.daysInTaxYear - daysPassed) * 100

  $: projectedAllowableExpenses = allowableExpenses * 100 / percentageComplete

  $: totalIncome = Math.floor(showProjection
    ? (taxYear.daysInTaxYear * (percentageWorked / 100)) * dayRate
    : daysWorked * dayRate)

  $: netProfit = Math.max(
    0,
    totalIncome - Math.ceil(showProjection ? projectedAllowableExpenses : allowableExpenses)
  )

  $: taxableIncome = Math.max(0, netProfit - taxYear.personalAllowance)

  $: incomeTaxBand1 = Math.min(
    taxableIncome,
    taxYear.incomeTaxBand2Threshold
  ) * (taxYear.incomeTaxBand1Rate / 100)

  $: incomeTaxBand2 = Math.min(
    Math.max(0, taxableIncome - taxYear.incomeTaxBand2Threshold),
    taxYear.incomeTaxBand3Threshold - taxYear.incomeTaxBand2Threshold
  ) * (taxYear.incomeTaxBand2Rate / 100)

  $: incomeTaxBand3 = Math.max(
    taxableIncome - taxYear.incomeTaxBand3Threshold,
    0
  ) * (taxYear.incomeTaxBand3Rate / 100)

  $: incomeTax = incomeTaxBand1 + incomeTaxBand2 + incomeTaxBand3

  $: nic2 = showProjection
    ? taxYear.nic2WeeklyRate * Math.ceil(taxYear.daysInTaxYear / 7)
    : taxYear.nic2WeeklyRate * weeksPassed

  $: nic4Band1 = Math.min(
    netProfit,
    taxYear.nic4Band2Threshold
  ) * (taxYear.nic4Band1Rate / 100)

  $: nic4Band2 = Math.min(
    Math.max(0, netProfit - taxYear.nic4Band2Threshold),
    taxYear.nic4Band3Threshold - taxYear.nic4Band2Threshold
  ) * (taxYear.nic4Band2Rate / 100)

  $: nic4Band3 = Math.max(
    netProfit - taxYear.nic4Band3Threshold,
    0
  ) * (taxYear.nic4Band3Rate / 100)

  $: nic4 = nic4Band1 + nic4Band2 + nic4Band3

  $: totalTax = incomeTax + nic2 + nic4

  const currency = new Intl.NumberFormat('en-GB', {
    style: 'currency',
    currency: 'GBP',
    minimumFractionDigits: 2,
  })
</script>

<style>
  main {
    max-width: 800px;
    padding: 40px 20px;
    width: 100%;
  }

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

<main>
  <h1>UK Tax Year</h1>
  <select on:change={e => selectedTaxYear = e.target.value}>
    {#each taxYears as year, i}
      <option selected={i === selectedTaxYear} value={i}>
        {year.taxYearStartDate} - {year.taxYearEndDate}
      </option>
    {/each}
  </select>
  {#if taxYear.taxYearEnded}
    <p>This tax year has ended</p>
  {/if}

  <hr />

  <form>
    {#if !taxYear.taxYearEnded}
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
    {/if}

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

    {#if !taxYear.taxYearEnded}
      <div>
        <input type="checkbox" bind:checked={includeToday} />
        <label>Include today?</label>
      </div>
    {/if}

    {#if !taxYear.taxYearEnded && showProjection}
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

    {#if dayRate && !taxYear.taxYearEnded}
      {#if percentageWorked > percentageRemaining}
        <p>
          <strong>Warning:</strong> Projected to pass the VAT threshold of
          {currency.format(taxYear.vatThreshold)}. You can work another
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
  <p>Allowable expenses: {currency.format(Math.ceil(allowableExpenses))}</p>
  <p>Net profit: {currency.format(netProfit)}</p>
  <p>Taxable income: {currency.format(taxableIncome)}</p>

  <hr />

  <p>
    Income tax band 1 ({taxYear.incomeTaxBand1Rate}%):
    {currency.format(incomeTaxBand1)}
  </p>

  <p>
    Income tax band 2 ({taxYear.incomeTaxBand2Rate}%):
    {currency.format(incomeTaxBand2)}
  </p>

  <p>
    Income tax band 3 ({taxYear.incomeTaxBand3Rate}%):
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
    NIC (class 4) band 1 ({taxYear.nic4Band1Rate}%):
    {currency.format(nic4Band1)}
  </p>

  <p>
    NIC (class 4) band 2 ({taxYear.nic4Band2Rate}%):
    {currency.format(nic4Band2)}
  </p>

  <p>
    NIC (class 4) band 3 ({taxYear.nic4Band3Rate}%):
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

  <hr />

  <p>
    Income after tax:
    {currency.format(totalIncome - totalTax)}
  </p>
</main>