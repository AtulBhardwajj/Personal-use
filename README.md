# Personal-use
Important 

import { test, expect, Page } from '@playwright/test';

const BASE_URL = 'https://opensource-demo.orangehrmlive.com';
const ADMIN_USER = 'Admin';
const ADMIN_PASS = 'admin123';

// Unique values so the test can run repeatedly
const stamp = Date.now().toString().slice(-6);
const employee = {
  firstName: 'John',
  middleName: 'A',
  lastName: `Doe${stamp}`,
  username: `john.doe${stamp}`,
  password: 'Test@12345',
};

// Find an input inside an OrangeHRM field group by its label text
const fieldByLabel = (page: Page, label: string) =>
  page
    .locator('.oxd-input-group')
    .filter({ has: page.locator('label', { hasText: new RegExp(`^${label}$`) }) });

async function fillText(page: Page, label: string, value: string) {
  await fieldByLabel(page, label).locator('input, textarea').first().fill(value);
}

async function selectDropdown(page: Page, label: string, option: string) {
  await fieldByLabel(page, label).locator('.oxd-select-text').click();
  await page.getByRole('option', { name: option, exact: true }).click();
}

test('OrangeHRM - add employee with login, personal and contact details', async ({ page }) => {
  // 1. Login
  await page.goto(`${BASE_URL}/web/index.php/auth/login`);
  await page.locator('input[name="username"]').fill(ADMIN_USER);
  await page.locator('input[name="password"]').fill(ADMIN_PASS);
  await page.getByRole('button', { name: 'Login' }).click();
  await expect(page).toHaveURL(/dashboard/);

  // 2. Go to PIM -> Add Employee
  await page.getByRole('link', { name: 'PIM' }).click();
  await page.getByRole('link', { name: 'Add Employee' }).click();
  await expect(page.getByRole('heading', { name: 'Add Employee' })).toBeVisible();

  // 3. Full name
  await page.locator('input[name="firstName"]').fill(employee.firstName);
  await page.locator('input[name="middleName"]').fill(employee.middleName);
  await page.locator('input[name="lastName"]').fill(employee.lastName);

  // 4. Create Login Details
  await page.locator('.oxd-switch-input').click();
  await fillText(page, 'Username', employee.username);
  await fillText(page, 'Password', employee.password);
  await fillText(page, 'Confirm Password', employee.password);

  // 5. Save
  await page.getByRole('button', { name: 'Save' }).click();
  await expect(page.getByText('Successfully Saved')).toBeVisible();

  // 6. Personal Details
  await expect(page).toHaveURL(/viewPersonalDetails/);
  await expect(page.getByRole('heading', { name: 'Personal Details' })).toBeVisible();
  // wait for the form to finish loading
  await expect(page.locator('input[name="firstName"]')).toHaveValue(employee.firstName);

  await fillText(page, "Driver's License Number", 'DL1234567');
  await fillText(page, 'License Expiry Date', '2030-25-12'); // yyyy-dd-mm (demo default format)
  await page.keyboard.press('Escape'); // close date picker

  await selectDropdown(page, 'Nationality', 'Indian');
  await selectDropdown(page, 'Marital Status', 'Single');

  await fillText(page, 'Date of Birth', '1995-15-05');
  await page.keyboard.press('Escape');
  await page.getByText('Male', { exact: true }).click(); // gender radio

  // Save the Personal Details form (first Save button on the page)
  await page.locator('form').filter({ hasText: 'Employee Full Name' })
    .getByRole('button', { name: 'Save' }).click();
  await expect(page.getByText('Successfully Updated')).toBeVisible();

  // 7. Contact Details
  await page.getByRole('link', { name: 'Contact Details' }).click();
  await expect(page.getByRole('heading', { name: 'Contact Details' })).toBeVisible();

  await fillText(page, 'Street 1', '123 Main Street');
  await fillText(page, 'Street 2', 'Near City Park');
  await fillText(page, 'City', 'Mumbai');
  await fillText(page, 'State/Province', 'Maharashtra');
  await fillText(page, 'Zip/Postal Code', '400001');
  await selectDropdown(page, 'Country', 'India');
  await fillText(page, 'Home', '0221234567');
  await fillText(page, 'Mobile', '9876543210');
  await fillText(page, 'Work', '0227654321');
  await fillText(page, 'Work Email', `john.work${stamp}@example.com`);
  await fillText(page, 'Other Email', `john.other${stamp}@example.com`);

  await page.getByRole('button', { name: 'Save' }).click();
  await expect(page.getByText('Successfully Updated')).toBeVisible();
});

