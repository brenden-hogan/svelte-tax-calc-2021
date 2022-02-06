<script>
  let income = localStorage.getItem("income") || 32000;
  let filingStatus = localStorage.getItem("filingStatus") || "single";
  $: federalTaxDue = getFederalTaxResponsibility(
    income,
    getBracketMap(filingStatus)
  );
  $: federalTaxDueDisplay = formatMoney(federalTaxDue);
  $: federalTaxRate = ((federalTaxDue / income) * 100).toFixed(2);

  $: ssTaxDue = income * (6.2 / 100);
  $: ssTaxDueDisplay = formatMoney(ssTaxDue);
  $: ssTaxRate = ((ssTaxDue / income) * 100).toFixed(2);

  $: mcTaxDue = income * (1.45 / 100);
  $: mcTaxDueDisplay = formatMoney(mcTaxDue);
  $: mcTaxRate = ((mcTaxDue / income) * 100).toFixed(2);

  $: stateTaxDue = income * (4.55 / 100);
  $: stateTaxDueDisplay = formatMoney(stateTaxDue);
  $: stateTaxRate = ((stateTaxDue / income) * 100).toFixed(2);

  $: totalTaxDue = +federalTaxDue + +ssTaxDue + +mcTaxDue + +stateTaxDue;
  $: totalTaxDueDisplay = formatMoney(totalTaxDue);
  $: totalTaxRate = ((totalTaxDue / income) * 100).toFixed(2);

  const marriedJoint2021Brackets = new Map([
    [19900, 10],
    [81050, 12],
    [172750, 22],
    [329850, 24],
    [418850, 32],
    [628300, 35],
    [1000000000000000000000000000000, 37],
  ]);

  const marriedSeperate2021Brackets = new Map([
    [9950, 10],
    [40525, 12],
    [86375, 22],
    [164925, 24],
    [209425, 32],
    [314150, 35],
    [1000000000000000000000000000000, 37],
  ]);

  const headOfHousehold2021Brackets = new Map([
    [14200, 10],
    [54200, 12],
    [86350, 22],
    [164900, 24],
    [209400, 32],
    [523600, 35],
    [1000000000000000000000000000000, 37],
  ]);

  const single2021Brackets = new Map([
    [9950, 10],
    [40525, 12],
    [86375, 22],
    [164925, 24],
    [209425, 32],
    [523600, 35],
    [1000000000000000000000000000000, 37],
  ]);

  const getBracketMap = (bracketString) => {
    if (bracketString === "single") {
      return single2021Brackets;
    } else if (bracketString === "marriedJoint") {
      return marriedJoint2021Brackets;
    } else if (bracketString === "marriedSeperate") {
      return marriedSeperate2021Brackets;
    } else if (bracketString === "headOfHousehold") {
      return headOfHousehold2021Brackets;
    }
    return null;
  };

  const formatMoney = (
    amount,
    decimalCount = 2,
    decimal = ".",
    thousands = ","
  ) => {
    try {
      decimalCount = Math.abs(decimalCount);
      decimalCount = isNaN(decimalCount) ? 2 : decimalCount;

      const negativeSign = amount < 0 ? "-" : "";

      let i = parseInt(
        (amount = Math.abs(Number(amount) || 0).toFixed(decimalCount))
      ).toString();
      let j = i.length > 3 ? i.length % 3 : 0;

      return (
        negativeSign +
        (j ? i.substr(0, j) + thousands : "") +
        i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + thousands) +
        (decimalCount
          ? decimal +
            Math.abs(amount - i)
              .toFixed(decimalCount)
              .slice(2)
          : "")
      );
    } catch (e) {
      console.log(e);
    }
  };

  const getFederalTaxResponsibility = (incomeValue, brackets) => {
    let taxSum = 0;
    let prevLevel = 0;
    incomeValue = incomeValue - 24800; //Standard Deduction
    console.log(incomeValue);
    console.log(brackets);
    for (const [key, value] of brackets.entries()) {
      if (incomeValue <= key) {
        taxSum = (incomeValue - prevLevel) * (value / 100) + taxSum;
        if (taxSum <= 0) {
          return 0;
        } else {
          return taxSum;
        }
      } else {
        taxSum = (key - prevLevel) * (value / 100) + taxSum;
        prevLevel = key;
      }
    }
    return 0;
  };

  const setLocalStorage = () => {
    localStorage.setItem("filingStatus", filingStatus);
    localStorage.setItem("income", income);
  };
</script>

<main>
  <h1>
    Your Federal Income Tax Owed is: ${federalTaxDueDisplay} which is marginal tax
    rate of: {federalTaxRate}%
  </h1>

  <hr />

  <div class="block">
    <label for="incomeInput">Income: </label>
    <input
      type="number"
      bind:value={income}
      min="0"
      max="1000000000000000000000000000000"
      on:change={setLocalStorage}
    />
  </div>

  <div class="block">
    <label for="filingStatus">Choose a filing status: </label>
    <select
      name="status"
      id="filingStatus"
      bind:value={filingStatus}
      on:change={setLocalStorage}
    >
      <option value="single">Single</option>
      <option value="marriedJoint">Married Filing Joint</option>
      <option value="marriedSeperate">Married Filing Seperate</option>
      <option value="headOfHousehold">Head of Household</option>
    </select>
  </div>

  <div class="block">
    <table>
      <tr>
        <th>Tax Source</th>
        <th>Amount</th>
        <th>Percent</th>
      </tr>
      <tr>
        <th>Federal</th>
        <th>${federalTaxDueDisplay}</th>
        <th>{federalTaxRate}%</th>
      </tr>
      <tr>
        <th>Social Security</th>
        <th>${ssTaxDueDisplay}</th>
        <th>{ssTaxRate}%</th>
      </tr>
      <tr>
        <th>Medicare</th>
        <th>${mcTaxDueDisplay}</th>
        <th>{mcTaxRate}%</th>
      </tr>
      <tr>
        <th>State (CO)</th>
        <th>${stateTaxDueDisplay}</th>
        <th>{stateTaxRate}%</th>
      </tr>
      <tr>
        <th>Total</th>
        <th>${totalTaxDueDisplay}</th>
        <th>{totalTaxRate}%</th>
      </tr>
    </table>
  </div>
</main>

<style>
  label {
    text-align: left;
    display: inline-block;
  }
  div {
    text-align: left;
    padding: 0.5em;
    margin: 0.5em;
    background-color: black;
    color: #4d54b3;
    border: "#4d54b3S";
  }
</style>
