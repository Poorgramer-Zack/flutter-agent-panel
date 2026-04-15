---
name: shadcn-flutter-components
description: Detailed component usage examples and API documentation for Shadcn UI Flutter.
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# Shadcn UI Components Reference

Complete API reference for all Shadcn UI Flutter components.

## Table of Contents

- [Accordion](#accordion)
- [Alert](#alert)
- [Avatar](#avatar)
- [Badge](#badge)
- [Button](#button)
- [Calendar](#calendar)
- [Card](#card)
- [Checkbox](#checkbox)
- [ContextMenu](#contextmenu)
- [DatePicker](#datepicker)
- [Dialog](#dialog)
- [Form](#form)
- [IconButton](#iconbutton)
- [Input](#input)
- [InputOTP](#inputotp)
- [Menubar](#menubar)
- [Popover](#popover)
- [Progress](#progress)
- [RadioGroup](#radiogroup)
- [Resizable](#resizable)
- [Select](#select)
- [Separator](#separator)
- [Sheet](#sheet)
- [Slider](#slider)
- [Sonner](#sonner)
- [Switch](#switch)
- [Table](#table)
- [Tabs](#tabs)
- [Textarea](#textarea)
- [TimePicker](#timepicker)
- [Toast](#toast)
- [Tooltip](#tooltip)

---

## Accordion

Vertically stacked interactive headings that reveal content.

### Single Selection

```dart
final details = [
  (title: 'Is it acceptable?', content: 'Yes. It adheres to the WAI-ARIA design pattern.'),
  (title: 'Is it styled?', content: 'Yes. It comes with default styles.'),
];

ShadAccordion<({String content, String title})>(
  children: details.map(
    (detail) => ShadAccordionItem(
      value: detail,
      title: Text(detail.title),
      child: Text(detail.content),
    ),
  ),
)
```

### Multiple Selection

```dart
ShadAccordion<({String content, String title})>.multiple(
  children: details.map(
    (detail) => ShadAccordionItem(
      value: detail,
      title: Text(detail.title),
      child: Text(detail.content),
    ),
  ),
)
```

---

## Alert

Displays a callout for user attention.

### Default

```dart
ShadAlert(
  icon: Icon(LucideIcons.terminal),
  title: Text('Heads up!'),
  description: Text('You can add components to your app using the cli.'),
)
```

### Destructive

```dart
ShadAlert.destructive(
  icon: Icon(LucideIcons.circleAlert),
  title: Text('Error'),
  description: Text('Your session has expired. Please log in again.'),
)
```

---

## Avatar

An image element with a placeholder for representing the user.

```dart
ShadAvatar(
  'https://avatars.githubusercontent.com/u/124599?v=4',
  placeholder: Text('CN'),
)
```

---

## Badge

Displays a badge or a component that looks like a badge.

### Variants

```dart
ShadBadge(child: const Text('Primary'))
ShadBadge.secondary(child: const Text('Secondary'))
ShadBadge.destructive(child: const Text('Destructive'))
ShadBadge.outline(child: const Text('Outline'))
```

---

## Button

Displays a button or a component that looks like a button.

### Variants

```dart
// Primary
ShadButton(
  child: const Text('Primary'),
  onPressed: () {},
)

// Secondary
ShadButton.secondary(
  child: const Text('Secondary'),
  onPressed: () {},
)

// Destructive
ShadButton.destructive(
  child: const Text('Destructive'),
  onPressed: () {},
)

// Outline
ShadButton.outline(
  child: const Text('Outline'),
  onPressed: () {},
)

// Ghost
ShadButton.ghost(
  child: const Text('Ghost'),
  onPressed: () {},
)

// Link
ShadButton.link(
  child: const Text('Link'),
  onPressed: () {},
)
```

### With Icon

```dart
ShadButton(
  onPressed: () {},
  leading: const Icon(LucideIcons.mail),
  child: const Text('Login with Email'),
)
```

### Loading State

```dart
ShadButton(
  onPressed: () {},
  leading: SizedBox.square(
    dimension: 16,
    child: CircularProgressIndicator(
      strokeWidth: 2,
      color: ShadTheme.of(context).colorScheme.primaryForeground,
    ),
  ),
  child: const Text('Please wait'),
)
```

### With Gradient and Shadow

```dart
ShadButton(
  onPressed: () {},
  gradient: const LinearGradient(colors: [Colors.cyan, Colors.indigo]),
  shadows: [
    BoxShadow(
      color: Colors.blue.withOpacity(.4),
      spreadRadius: 4,
      blurRadius: 10,
      offset: const Offset(0, 2),
    ),
  ],
  child: const Text('Gradient with Shadow'),
)
```

---

## Calendar

A date field component that allows users to enter and edit date.

### Single

```dart
final today = DateTime.now();

ShadCalendar(
  selected: today,
  fromMonth: DateTime(today.year - 1),
  toMonth: DateTime(today.year, 12),
)
```

### Multiple

```dart
ShadCalendar.multiple(
  numberOfMonths: 2,
  fromMonth: DateTime(today.year),
  toMonth: DateTime(today.year + 1, 12),
  min: 5,
  max: 10,
)
```

### Range

```dart
ShadCalendar.range(
  min: 2,
  max: 5,
)
```

### Caption Layout Options

```dart
ShadCalendar(
  captionLayout: ShadCalendarCaptionLayout.dropdownMonths,
)

ShadCalendar(
  captionLayout: ShadCalendarCaptionLayout.dropdownYears,
)
```

### Options

```dart
ShadCalendar(
  hideNavigation: true,          // Hide navigation
  showWeekNumbers: true,         // Show week numbers
  showOutsideDays: false,        // Hide outside days
  fixedWeeks: true,              // Fixed weeks
  hideWeekdayNames: true,        // Hide weekday names
)
```

---

## Card

Displays a card with header, content, and footer.

```dart
ShadCard(
  width: 350,
  title: Text('Create project', style: theme.textTheme.h4),
  description: const Text('Deploy your new project in one-click.'),
  footer: Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: [
      ShadButton.outline(
        child: const Text('Cancel'),
        onPressed: () {},
      ),
      ShadButton(
        child: const Text('Deploy'),
        onPressed: () {},
      ),
    ],
  ),
  child: const Text('Card content'),
)
```

---

## Checkbox

A control that allows the user to toggle between checked and not checked.

```dart
class CheckboxExample extends StatefulWidget {
  @override
  State createState() => _CheckboxExampleState();
}

class _CheckboxExampleState extends State {
  bool value = false;

  @override
  Widget build(BuildContext context) {
    return ShadCheckbox(
      value: value,
      onChanged: (v) => setState(() => value = v),
      label: const Text('Accept terms and conditions'),
      sublabel: const Text('You agree to our Terms of Service and Privacy Policy.'),
    );
  }
}
```

### Form Field

```dart
ShadCheckboxFormField(
  id: 'terms',
  initialValue: false,
  inputLabel: const Text('I accept the terms and conditions'),
  inputSublabel: const Text('You agree to our Terms and Conditions'),
  validator: (v) {
    if (!v) return 'You must accept the terms and conditions';
    return null;
  },
)
```

---

## ContextMenu

Displays a menu triggered by mouse right-click.

```dart
ShadContextMenuRegion(
  constraints: const BoxConstraints(minWidth: 300),
  items: [
    const ShadContextMenuItem.inset(child: Text('Back')),
    const ShadContextMenuItem.inset(enabled: false, child: Text('Forward')),
    const ShadContextMenuItem.inset(child: Text('Reload')),
    const Divider(height: 8),
    ShadContextMenuItem(
      leading: Icon(LucideIcons.check),
      child: Text('Show Bookmarks Bar'),
    ),
  ],
  child: Container(
    width: 300,
    height: 200,
    alignment: Alignment.center,
    child: const Text('Right click here'),
  ),
)
```

---

## DatePicker

A date picker component with range and presets.

### Single Date Picker

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 600),
  child: const ShadDatePicker(),
)
```

### Date Range Picker

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 600),
  child: const ShadDatePicker.range(),
)
```

### With Presets

```dart
const presets = {
  0: 'Today',
  1: 'Tomorrow',
  7: 'In a week',
};

ShadDatePicker(
  header: ShadSelect(
    placeholder: const Text('Select'),
    options: presets.entries
      .map((e) => ShadOption(value: e.key, child: Text(e.value)))
      .toList(),
    selectedOptionBuilder: (context, value) => Text(presets[value]!),
    onChanged: (value) {
      if (value == null) return;
      setState(() {
        selected = DateTime.now().add(Duration(days: value));
      });
    },
  ),
  selected: selected,
)
```

### Form Field

```dart
ShadDatePickerFormField(
  label: const Text('Date of birth'),
  onChanged: print,
  description: const Text('Your date of birth is used to calculate your age.'),
  validator: (v) {
    if (v == null) return 'A date of birth is required.';
    return null;
  },
)
```

### Date Range Form Field

```dart
ShadDateRangePickerFormField(
  label: const Text('Range of dates'),
  onChanged: print,
  validator: (v) {
    if (v == null) return 'A range of dates is required.';
    if (v.start == null) return 'The start date is required.';
    if (v.end == null) return 'The end date is required.';
    return null;
  },
)
```

---

## Dialog

A modal dialog that interrupts the user.

### Basic Dialog

```dart
ShadButton.outline(
  child: const Text('Edit Profile'),
  onPressed: () {
    showShadDialog(
      context: context,
      builder: (context) => ShadDialog(
        title: const Text('Edit Profile'),
        description: const Text("Make changes to your profile here. Click save when you're done"),
        actions: const [ShadButton(child: Text('Save changes'))],
        child: ContentWidget(),
      ),
    );
  },
)
```

### Alert Dialog

```dart
showShadDialog(
  context: context,
  builder: (context) => ShadDialog.alert(
    title: const Text('Are you absolutely sure?'),
    description: const Text('This action cannot be undone.'),
    actions: [
      ShadButton.outline(
        child: const Text('Cancel'),
        onPressed: () => Navigator.of(context).pop(false),
      ),
      ShadButton(
        child: const Text('Continue'),
        onPressed: () => Navigator.of(context).pop(true),
      ),
    ],
  ),
)
```

---

## Form

Builds a form with validation and easy access to form field values.

```dart
final formKey = GlobalKey<ShadFormState>();

ShadForm(
  key: formKey,
  child: Column(
    children: [
      ShadInputFormField(
        id: 'username',
        label: const Text('Username'),
        placeholder: const Text('Enter your username'),
        description: const Text('This is your public display name.'),
        validator: (v) {
          if (v.length < 2) {
            return 'Username must be at least 2 characters.';
          }
          return null;
        },
      ),
      ShadButton(
        child: const Text('Submit'),
        onPressed: () {
          if (formKey.currentState!.saveAndValidate()) {
            print('Validation succeeded: ${formKey.currentState!.value}');
          }
        },
      ),
    ],
  ),
)
```

---

## IconButton

Displays an icon button or a component that looks like a button with an icon.

### Variants

```dart
ShadIconButton(
  onPressed: () => print('Primary'),
  icon: const Icon(LucideIcons.rocket),
)

ShadIconButton.secondary(
  icon: const Icon(LucideIcons.rocket),
  onPressed: () => print('Secondary'),
)

ShadIconButton.destructive(
  icon: const Icon(LucideIcons.rocket),
  onPressed: () => print('Destructive'),
)

ShadIconButton.outline(
  icon: const Icon(LucideIcons.rocket),
  onPressed: () => print('Outline'),
)

ShadIconButton.ghost(
  icon: const Icon(LucideIcons.rocket),
  onPressed: () => print('Ghost'),
)
```

### Loading

```dart
ShadIconButton(
  icon: SizedBox.square(
    dimension: 16,
    child: CircularProgressIndicator(
      strokeWidth: 2,
      color: ShadTheme.of(context).colorScheme.primaryForeground,
    ),
  ),
)
```

### With Gradient and Shadow

```dart
ShadIconButton(
  gradient: const LinearGradient(colors: [Colors.cyan, Colors.indigo]),
  shadows: [
    BoxShadow(
      color: Colors.blue.withOpacity(.4),
      spreadRadius: 4,
      blurRadius: 10,
      offset: const Offset(0, 2),
    ),
  ],
  icon: const Icon(LucideIcons.rocket),
)
```

---

## Input

Displays a form input field or a component that looks like an input field.

### Basic Input

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 320),
  child: const ShadInput(
    placeholder: Text('Email'),
    keyboardType: TextInputType.emailAddress,
  ),
)
```

### With Leading and Trailing

```dart
ShadInput(
  placeholder: const Text('Password'),
  obscureText: obscure,
  leading: const Padding(
    padding: EdgeInsets.all(4.0),
    child: Icon(LucideIcons.lock),
  ),
  trailing: ShadButton(
    width: 24,
    height: 24,
    padding: EdgeInsets.zero,
    icon: Icon(obscure ? LucideIcons.eyeOff : LucideIcons.eye),
    onPressed: () => setState(() => obscure = !obscure),
  ),
)
```

### Form Field

```dart
ShadInputFormField(
  id: 'username',
  label: const Text('Username'),
  placeholder: const Text('Enter your username'),
  description: const Text('This is your public display name.'),
  validator: (v) {
    if (v.length < 2) {
      return 'Username must be at least 2 characters.';
    }
    return null;
  },
)
```

---

## InputOTP

Accessible one-time password component with copy paste functionality.

```dart
ShadInputOTP(
  onChanged: (v) => print('OTP: $v'),
  maxLength: 6,
  children: const [
    ShadInputOTPGroup(
      children: [
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
      ],
    ),
    Icon(size: 24, LucideIcons.dot),
    ShadInputOTPGroup(
      children: [
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
      ],
    ),
  ],
)
```

### With InputFormatters

```dart
ShadInputOTP(
  onChanged: (v) => print('OTP: $v'),
  maxLength: 4,
  keyboardType: TextInputType.number,
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,
  ],
  children: const [
    ShadInputOTPGroup(
      children: [
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
      ],
    ),
  ],
)
```

### Form Field

```dart
ShadInputOTPFormField(
  id: 'otp',
  maxLength: 6,
  label: const Text('OTP'),
  description: const Text('Enter your OTP.'),
  validator: (v) {
    if (v.contains(' ')) {
      return 'Fill the whole OTP code';
    }
    return null;
  },
  children: const [
    ShadInputOTPGroup(
      children: [
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
      ],
    ),
    Icon(size: 24, LucideIcons.dot),
    ShadInputOTPGroup(
      children: [
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
        ShadInputOTPSlot(),
      ],
    ),
  ],
)
```

---

## Menubar

A visually persistent menu common in desktop applications.

```dart
ShadMenubar(
  items: [
    ShadMenubarItem(
      items: [
        const ShadContextMenuItem(child: Text('New Tab')),
        const ShadContextMenuItem(child: Text('New Window')),
        Divider(height: 8),
        const ShadContextMenuItem(child: Text('Print...')),
      ],
      child: const Text('File'),
    ),
    ShadMenubarItem(
      items: [
        const ShadContextMenuItem(child: Text('Undo')),
        const ShadContextMenuItem(child: Text('Redo')),
        Divider(height: 8),
        const ShadContextMenuItem(child: Text('Cut')),
        const ShadContextMenuItem(child: Text('Copy')),
        const ShadContextMenuItem(child: Text('Paste')),
      ],
      child: const Text('Edit'),
    ),
  ],
)
```

---

## Popover

Displays rich content in a portal, triggered by a button.

```dart
class PopoverExample extends StatefulWidget {
  @override
  State createState() => _PopoverExampleState();
}

class _PopoverExampleState extends State {
  final popoverController = ShadPopoverController();

  @override
  void dispose() {
    popoverController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ShadPopover(
          controller: popoverController,
          popover: (context) => SizedBox(
            width: 288,
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text('Dimensions', style: ShadTheme.of(context).textTheme.h4),
                ShadInput(initialValue: '100%'),
              ],
            ),
          ),
          child: ShadButton.outline(
            onPressed: popoverController.toggle,
            child: const Text('Open popover'),
          ),
        ),
      ),
    );
  }
}
```

---

## Progress

Displays an indicator showing the completion progress of a task.

### Determinate

```dart
ConstrainedBox(
  constraints: BoxConstraints(maxWidth: MediaQuery.sizeOf(context).width * 0.6),
  child: const ShadProgress(value: 0.5),
)
```

### Indeterminate

```dart
ConstrainedBox(
  constraints: BoxConstraints(maxWidth: MediaQuery.sizeOf(context).width * 0.6),
  child: const ShadProgress(),
)
```

---

## RadioGroup

A set of checkable buttons where no more than one can be checked at a time.

```dart
ShadRadioGroup(
  items: [
    ShadRadio(
      label: Text('Default'),
      value: 'default',
    ),
    ShadRadio(
      label: Text('Comfortable'),
      value: 'comfortable',
    ),
    ShadRadio(
      label: Text('Nothing'),
      value: 'nothing',
    ),
  ],
)
```

### Form Field

```dart
enum NotifyAbout { all, mentions, nothing; }

ShadRadioGroupFormField(
  label: const Text('Notify me about'),
  items: NotifyAbout.values.map(
    (e) => ShadRadio(
      value: e,
      label: Text(e.name),
    ),
  ),
  validator: (v) {
    if (v == null) return 'You need to select a notification type.';
    return null;
  },
)
```

---

## Resizable

Resizable panel groups and layouts.

### Basic

```dart
DecoratedBox(
  decoration: BoxDecoration(
    borderRadius: theme.radius,
    border: Border.all(color: theme.colorScheme.border),
  ),
  child: ClipRRect(
    borderRadius: theme.radius,
    child: ShadResizablePanelGroup(
      children: [
        ShadResizablePanel(
          id: 0,
          defaultSize: .5,
          minSize: .2,
          maxSize: .8,
          child: Center(child: Text('One')),
        ),
        ShadResizablePanel(
          id: 1,
          defaultSize: .5,
          child: Center(child: Text('Two')),
        ),
      ],
    ),
  ),
)
```

### Vertical

```dart
ShadResizablePanelGroup(
  axis: Axis.vertical,
  children: [
    ShadResizablePanel(
      id: 0,
      defaultSize: 0.3,
      minSize: 0.1,
      child: Center(child: Text('Header')),
    ),
    ShadResizablePanel(
      id: 1,
      defaultSize: 0.7,
      minSize: 0.1,
      child: Center(child: Text('Footer')),
    ),
  ],
)
```

### With Handle

```dart
ShadResizablePanelGroup(
  showHandle: true,
  children: [
    ShadResizablePanel(
      id: 0,
      defaultSize: .5,
      minSize: .2,
      child: Center(child: Text('Sidebar')),
    ),
    ShadResizablePanel(
      id: 1,
      defaultSize: .5,
      minSize: .2,
      child: Center(child: Text('Content')),
    ),
  ],
)
```

---

## Select

Displays a list of options for the user to pick from.

### Basic

```dart
final fruits = {
  'apple': 'Apple',
  'banana': 'Banana',
  'blueberry': 'Blueberry',
};

ShadSelect(
  placeholder: const Text('Select a fruit'),
  options: fruits.entries
    .map((e) => ShadOption(value: e.key, child: Text(e.value)))
    .toList(),
  selectedOptionBuilder: (context, value) => Text(fruits[value]!),
  onChanged: print,
)
```

### With Search

```dart
ShadSelect.withSearch(
  minWidth: 180,
  maxWidth: 300,
  placeholder: const Text('Select framework...'),
  onSearchChanged: (value) => setState(() => searchValue = value),
  searchPlaceholder: const Text('Search framework'),
  options: frameworks.entries
    .map((f) => ShadOption(value: f.key, child: Text(f.value)))
    .toList(),
  selectedOptionBuilder: (context, value) => Text(frameworks[value]!),
)
```

### Multiple Selection

```dart
ShadSelect.multiple(
  minWidth: 340,
  onChanged: print,
  allowDeselection: true,
  closeOnSelect: false,
  placeholder: const Text('Select multiple fruits'),
  options: fruits.entries
    .map((e) => ShadOption(value: e.key, child: Text(e.value)))
    .toList(),
  selectedOptionsBuilder: (context, values) =>
    Text(values.map((v) => v.capitalize()).join(', ')),
)
```

### Form Field

```dart
ShadSelectFormField(
  id: 'email',
  minWidth: 350,
  initialValue: null,
  options: emails
    .map((email) => ShadOption(value: email, child: Text(email)))
    .toList(),
  selectedOptionBuilder: (context, value) => Text(value ?? 'Select email'),
  placeholder: const Text('Select a verified email'),
  validator: (v) {
    if (v == null) return 'Please select an email';
    return null;
  },
)
```

---

## Separator

Visually or semantically separates content.

```dart
// Horizontal
const ShadSeparator.horizontal(
  thickness: 4,
  margin: EdgeInsets.symmetric(horizontal: 20),
  radius: BorderRadius.all(Radius.circular(4)),
)

// Vertical
const ShadSeparator.vertical(
  thickness: 4,
  margin: EdgeInsets.symmetric(vertical: 20),
  radius: BorderRadius.all(Radius.circular(4)),
)
```

---

## Sheet

Extends the Dialog component to display content that complements the main content.

```dart
showShadSheet(
  side: ShadSheetSide.right,
  context: context,
  builder: (context) => ShadSheet(
    constraints: const BoxConstraints(maxWidth: 512),
    title: const Text('Edit Profile'),
    description: const Text("Make changes to your profile here."),
    child: Padding(
      padding: const EdgeInsets.symmetric(vertical: 20),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ShadInput(initialValue: 'John'),
          ShadInput(initialValue: 'Doe'),
        ],
      ),
    ),
    actions: const [ShadButton(child: Text('Save changes'))],
  ),
)
```

### Side Options

- `ShadSheetSide.top`
- `ShadSheetSide.right`
- `ShadSheetSide.bottom`
- `ShadSheetSide.left`

---

## Slider

An input where the user selects a value from within a given range.

```dart
ShadSlider(
  initialValue: 33,
  max: 100,
)
```

---

## Sonner

An opinionated toast component.

```dart
ShadButton.outline(
  child: const Text('Show Toast'),
  onPressed: () {
    final sonner = ShadSonner.of(context);
    final id = Random().nextInt(1000);
    sonner.show(
      ShadToast(
        id: id,
        title: const Text('Event has been created'),
        description: Text(DateTime.now().toString()),
        action: ShadButton(
          child: const Text('Undo'),
          onPressed: () => sonner.hide(id),
        ),
      ),
    );
  },
)
```

---

## Switch

A control that allows the user to toggle between checked and not checked.

```dart
class SwitchExample extends StatefulWidget {
  @override
  State createState() => _SwitchExampleState();
}

class _SwitchExampleState extends State {
  bool value = false;

  @override
  Widget build(BuildContext context) {
    return ShadSwitch(
      value: value,
      onChanged: (v) => setState(() => value = v),
      label: const Text('Airplane Mode'),
    );
  }
}
```

### Form Field

```dart
ShadSwitchFormField(
  id: 'terms',
  initialValue: false,
  inputLabel: const Text('I accept the terms and conditions'),
  inputSublabel: const Text('You agree to our Terms and Conditions'),
  validator: (v) {
    if (!v) return 'You must accept the terms and conditions';
    return null;
  },
)
```

---

## Table

A responsive table component.

### List Mode (Small Tables)

```dart
const invoices = [
  (invoice: "INV001", paymentStatus: "Paid", totalAmount: r"$250.00"),
  (invoice: "INV002", paymentStatus: "Pending", totalAmount: r"$150.00"),
];

ShadTable.list(
  header: const [
    ShadTableCell.header(child: Text('Invoice')),
    ShadTableCell.header(child: Text('Status')),
    ShadTableCell.header(child: Text('Amount')),
  ],
  children: invoices.map(
    (invoice) => [
      ShadTableCell(child: Text(invoice.invoice)),
      ShadTableCell(child: Text(invoice.paymentStatus)),
      ShadTableCell(child: Text(invoice.totalAmount)),
    ],
  ),
)
```

### Builder Mode (Large Tables)

```dart
ShadTable(
  columnCount: 4,
  rowCount: invoices.length,
  header: (context, column) {
    return ShadTableCell.header(
      child: Text(['Invoice', 'Status', 'Method', 'Amount'][column]),
    );
  },
  builder: (context, index) {
    final invoice = invoices[index.row];
    return ShadTableCell(
      child: Text([invoice.invoice, invoice.paymentStatus, invoice.paymentMethod, invoice.totalAmount][index.column]),
    );
  },
)
```

---

## Tabs

A set of layered sections of content displayed one at a time.

```dart
ShadTabs(
  value: 'account',
  tabBarConstraints: const BoxConstraints(maxWidth: 400),
  contentConstraints: const BoxConstraints(maxWidth: 400),
  tabs: [
    ShadTab(
      value: 'account',
      content: ShadCard(
        title: const Text('Account'),
        child: Column(
          children: [
            ShadInputFormField(label: const Text('Name'), initialValue: 'Ale'),
          ],
        ),
      ),
      child: const Text('Account'),
    ),
    ShadTab(
      value: 'password',
      content: ShadCard(
        title: const Text('Password'),
        child: Column(
          children: [
            ShadInputFormField(label: const Text('New password'), obscureText: true),
          ],
        ),
      ),
      child: const Text('Password'),
    ),
  ],
)
```

---

## Textarea

Displays a form textarea or a component that looks like a textarea.

### Basic

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 400),
  child: const ShadTextarea(
    placeholder: Text('Type your message here'),
  ),
)
```

### Form Field

```dart
ShadTextareaFormField(
  id: 'bio',
  label: const Text('Bio'),
  placeholder: const Text('Tell us a little bit about yourself'),
  description: const Text('You can @mention other users.'),
  validator: (v) {
    if (v.length < 10) return 'Bio must be at least 10 characters.';
    return null;
  },
)
```

---

## TimePicker

A time picker component.

### Basic

```dart
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 600),
  child: const ShadTimePicker(
    trailing: Padding(
      padding: EdgeInsets.only(left: 8, top: 14),
      child: Icon(LucideIcons.clock4),
    ),
  ),
)
```

### Form Field

```dart
ShadTimePickerFormField(
  label: const Text('Pick a time'),
  onChanged: print,
  description: const Text('The time of the day you want to pick'),
  validator: (v) => v == null ? 'A time is required' : null,
)
```

### Period Form Field

```dart
ShadTimePickerFormField.period(
  label: const Text('Pick a time'),
  onChanged: print,
  description: const Text('The time of the day you want to pick'),
  validator: (v) => v == null ? 'A time is required' : null,
)
```

---

## Toast

A succinct message that is displayed temporarily.

### Simple

```dart
ShadButton.outline(
  child: const Text('Show Toast'),
  onPressed: () {
    ShadToaster.of(context).show(
      const ShadToast(
        description: Text('Your message has been sent.'),
      ),
    );
  },
)
```

### With Title

```dart
ShadToaster.of(context).show(
  const ShadToast(
    title: Text('Uh oh! Something went wrong'),
    description: Text('There was a problem with your request'),
  ),
)
```

### With Action

```dart
ShadToaster.of(context).show(
  ShadToast(
    title: const Text('Scheduled: Catch up'),
    description: const Text('Friday, February 10, 2023 at 5:57 PM'),
    action: ShadButton.outline(
      child: const Text('Undo'),
      onPressed: () => ShadToaster.of(context).hide(),
    ),
  ),
)
```

### Destructive

```dart
final theme = ShadTheme.of(context);
ShadToaster.of(context).show(
  ShadToast.destructive(
    title: const Text('Uh oh! Something went wrong'),
    description: const Text('There was a problem with your request'),
    action: ShadButton.destructive(
      child: const Text('Try again'),
      decoration: ShadDecoration(
        border: ShadBorder.all(
          color: theme.colorScheme.destructiveForeground,
          width: 1,
        ),
      ),
      onPressed: () => ShadToaster.of(context).hide(),
    ),
  ),
)
```

---

## Tooltip

A popup that displays information when an element receives keyboard focus or the mouse hovers over it.

```dart
ShadTooltip(
  builder: (context) => const Text('Add to library'),
  child: ShadButton.outline(
    child: const Text('Hover/Focus'),
    onPressed: () {},
  ),
)
```

**Note**: Tooltip works on hover only if the child uses `ShadGestureDetector`. For custom widgets, wrap with `ShadGestureDetector`.
