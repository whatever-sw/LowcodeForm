# LowcodeForm - Node-Based Form Builder

A web-based low-code programming editor for creating custom forms with visual node-graph programming, built with Svelte 5.

![Initial View](https://github.com/user-attachments/assets/79b7e7f5-65d2-4b4a-985f-6df6c7502b81)

## Features

- **Visual Node Graph Editor**: Drag-and-drop interface for creating form logic
- **Multiple Input Types**: Text, Number, and Email input fields
- **Computed Fields**: Create calculated fields based on other form values using JavaScript expressions
- **Form Validation**: Add custom validation rules with real-time feedback
- **Live Form Preview**: See your form in action as you build it
- **Connection-Based Logic**: Connect nodes to define dependencies between fields

![Working Form](https://github.com/user-attachments/assets/c6ae907f-baba-481c-87aa-2862f0de6f4a)

## Getting Started

### Installation

```bash
npm install
```

### Development

```bash
npm run dev
```

Visit `http://localhost:5173` to see the application.

### Building for Production

```bash
npm run build
```

## How to Use

### Creating Form Fields

1. **Add Nodes**: Click on node types in the left sidebar to add them to the canvas:
   - **Text Input**: For text entry fields
   - **Number Input**: For numeric values
   - **Email Input**: For email addresses
   - **Computed Field**: For calculated values
   - **Validation**: For custom validation rules

2. **Configure Nodes**: 
   - Click on any node to select it
   - Use the Properties panel on the right to configure:
     - Label: Display name for the field
     - Expression (for computed fields): JavaScript expression to calculate value
     - Validation Rule (for validation nodes): JavaScript expression that returns true/false

3. **Connect Nodes**:
   - Click the **→** button on a source node to start a connection
   - Click the **←** button on a target node to complete the connection
   - Connections define data flow and dependencies

4. **Move Nodes**: Click and drag any node to reposition it on the canvas

5. **Delete**: 
   - Click the **×** button on a node to delete it
   - Click the red circle on a connection line to delete the connection

### Example: Creating a Form with Computed Field

1. Add a "Number Input" node and label it "Age"
2. Add another "Number Input" node and label it "Salary"
3. Add a "Computed Field" node and label it "Total Income"
4. Set the expression to: `Age * Salary`
5. Connect Age → Total Income
6. Connect Salary → Total Income
7. Fill in values in the form preview to see the computed result!

![Validation Example](https://github.com/user-attachments/assets/44c7ae71-79d5-4e61-ab32-c7cbf3f88fac)

### Example: Adding Validation

1. Add a "Validation" node and label it "Age must be positive"
2. Set the validation rule to: `value > 0`
3. Connect the Age field → Validation node
4. Try entering a negative number - the field will show a red border indicating validation failure

## Technical Details

### Built With

- **Svelte 5**: Modern reactive framework with runes
- **SvelteKit**: Full-stack framework for building web applications
- **Tailwind CSS**: Utility-first CSS framework
- **Vite**: Fast build tool

### Key Components

- **NodeGraphEditor.svelte**: Visual node graph editor with drag-and-drop
- **FormDisplay.svelte**: Live form preview with real-time computed fields and validation
- **+page.svelte**: Main application layout

### Architecture

The application uses Svelte 5's new runes system:
- `$state`: For reactive state management
- `$derived`: For computed values
- `$effect`: For side effects like evaluating expressions
- `$bindable`: For two-way data binding between components

Computed fields and validations are evaluated using JavaScript's `Function` constructor, allowing users to write arbitrary expressions that are evaluated in a controlled context with access to connected field values.

## License

MIT
