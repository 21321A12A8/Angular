import { Component } from '@angular/core';
import { FormBuilder, FormGroup, FormArray, Validators, AbstractControl } from '@angular/forms';

@Component({
  selector: 'app-root',
  template: `
    <h2>Dynamic Form Builder</h2>

    <form [formGroup]="configForm">
      <div formArrayName="fields">
        <div *ngFor="let field of fields.controls; let i = index" [formGroupName]="i" style="margin-bottom: 1rem; border: 1px solid #ccc; padding: 1rem;">
          <label>Label:
            <input formControlName="label" placeholder="Field Label" />
          </label>

          <label>Type:
            <select formControlName="type">
              <option value="text">Text</option>
              <option value="number">Number</option>
              <option value="email">Email</option>
              <option value="dropdown">Dropdown</option>
            </select>
          </label>

          <label>
            <input type="checkbox" formControlName="required" />
            Required
          </label>

          <div *ngIf="field.get('type')?.value === 'dropdown'">
            <label>Options (comma-separated):
              <input formControlName="options" placeholder="e.g. Option1,Option2" />
            </label>
          </div>

          <button (click)="removeField(i)" type="button">Remove</button>
        </div>
      </div>

      <button (click)="addField()" type="button">Add Field</button>
      <button (click)="generateForm()" type="button">Generate Form</button>
    </form>

    <div *ngIf="generatedForm">
      <h3>Generated Form</h3>
      <form [formGroup]="generatedForm" (ngSubmit)="submitForm()">
        <div *ngFor="let field of configForm.value.fields; let i = index">
          <div [ngSwitch]="field.type" style="margin-bottom: 1rem;">
            <label>{{ field.label }}</label>

            <input *ngSwitchCase="'text'" [formControlName]="field.label" type="text" />
            <input *ngSwitchCase="'number'" [formControlName]="field.label" type="number" />
            <input *ngSwitchCase="'email'" [formControlName]="field.label" type="email" />

            <select *ngSwitchCase="'dropdown'" [formControlName]="field.label">
              <option *ngFor="let opt of field.options.split(',')" [value]="opt.trim()">{{ opt.trim() }}</option>
            </select>

            <div style="color: red;" *ngIf="generatedForm.get(field.label)?.invalid && generatedForm.get(field.label)?.touched">
              {{ field.label }} is required.
            </div>
          </div>
        </div>

        <button type="submit" [disabled]="generatedForm.invalid">Submit</button>
      </form>

      <div *ngIf="submittedData">
        <h4>Submitted Data (JSON)</h4>
        <pre>{{ submittedData | json }}</pre>
      </div>
    </div>
  `,
})
export class AppComponent {
  configForm: FormGroup;
  generatedForm!: FormGroup;
  submittedData: any;

  constructor(private fb: FormBuilder) {
    this.configForm = this.fb.group({
      fields: this.fb.array([])
    });
    this.addField(); // Add one default field
  }

  get fields(): FormArray {
    return this.configForm.get('fields') as FormArray;
  }

  addField(): void {
    this.fields.push(this.fb.group({
      label: [''],
      type: ['text'],
      required: [false],
      options: ['']
    }));
  }

  removeField(index: number): void {
    this.fields.removeAt(index);
  }

  generateForm(): void {
    const formGroup: { [key: string]: AbstractControl } = {};
    for (let field of this.configForm.value.fields) {
      const validators = field.required ? [Validators.required] : [];
      formGroup[field.label] = this.fb.control('', validators);
    }
    this.generatedForm = this.fb.group(formGroup);
    this.submittedData = null;
  }

  submitForm(): void {
    if (this.generatedForm.valid) {
      this.submittedData = this.generatedForm.value;
    }
  }
}
