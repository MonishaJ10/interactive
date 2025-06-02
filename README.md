interactive dashboard comp ts
import { Component, Input, Output, EventEmitter, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { AgChartsModule } from 'ag-charts-angular';

@Component({
  selector: 'app-interactive-dashboard',
  standalone: true,
  imports: [CommonModule, FormsModule, AgChartsModule],
  templateUrl: './interactive-dashboard.component.html',
  styleUrls: ['./interactive-dashboard.component.css']
})
export class InteractiveDashboardComponent implements OnInit {
  @Input() editData: any;
  @Output() dashboardClose = new EventEmitter<void>();

  showModal = true;
  isFullscreen = false;
  isEditMode = false;
  currentStep = 0;
  steps = ['Initial', 'Content', 'Review'];

  formData = {
    name: '',
    description: '',
    isPublic: true
  };

  model: string = '';
  groupBy: string = '';
  aggregation: string = '';
  aggregationField: string = '';
  columnField: string = '';

  availableTemplates = ['BarChart', 'PieChart'];
  selectedTemplates: string[] = [];

  ngOnInit(): void {
    if (this.editData) {
      this.isEditMode = true;
      this.formData.name = this.editData.name;
      this.formData.description = this.editData.description;
      this.formData.isPublic = this.editData.isPublic;
      this.model = this.editData.model;
      this.groupBy = this.editData.groupBy;
      this.aggregation = this.editData.aggregation;
      this.aggregationField = this.editData.aggregationField;
      this.columnField = this.editData.columnField;
      this.selectedTemplates = this.editData.selectedTemplates || [];
    }
  }

  nextStep(): void {
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
    }
  }

  prevStep(): void {
    if (this.currentStep > 0) {
      this.currentStep--;
    }
  }

  toggleTemplate(template: string): void {
    if (!this.selectedTemplates.includes(template)) {
      this.selectedTemplates.push(template);
    }
  }

  removeTemplate(template: string): void {
    this.selectedTemplates = this.selectedTemplates.filter(t => t !== template);
  }

  toggleFullscreen(): void {
    this.isFullscreen = !this.isFullscreen;
  }

  submitDashboard(): void {
    const dashboardPayload = {
      name: this.formData.name,
      description: this.formData.description,
      isPublic: this.formData.isPublic,
      model: this.model,
      groupBy: this.groupBy,
      aggregation: this.aggregation,
      aggregationField: this.aggregationField,
      columnField: this.columnField,
      selectedTemplates: this.selectedTemplates,
      createdBy: this.editData?.createdBy || 'h59606',
      createdDate: this.editData?.createdDate || new Date().toISOString(),
      modifiedBy: 'h59606',
      modifiedDate: new Date().toISOString(),
    };

    // Make HTTP POST or PUT call here
    this.dashboardClose.emit();
  }

  closeModal(): void {
    this.dashboardClose.emit();
  }
}

interactive dashboard comp html
<div class="modal" [class.fullscreen]="isFullscreen">
  <div class="modal-header">
    <h2>{{ isEditMode ? 'Edit' : 'Create' }} Interactive Dashboard</h2>
    <button (click)="toggleFullscreen()">🗖</button>
    <button (click)="closeModal()">✖</button>
  </div>

  <div class="steps">
    <button *ngFor="let step of steps; let i = index"
            [class.active]="i === currentStep"
            (click)="currentStep = i">
      {{ step }}
    </button>
  </div>

  <div class="step-content" *ngIf="currentStep === 0">
    <label>Name:</label>
    <input [(ngModel)]="formData.name" type="text" />
    <label>Description:</label>
    <textarea [(ngModel)]="formData.description"></textarea>
    <label>
      <input type="checkbox" [(ngModel)]="formData.isPublic" />
      Public
    </label>
  </div>

  <div class="step-content" *ngIf="currentStep === 1">
    <label>Model:</label>
    <select [(ngModel)]="model">
      <option>Germany_Nostro</option>
      <option>Monaco_Nostro</option>
      <option>Switzerland_Nostro</option>
    </select>

    <label>Group By:</label>
    <select [(ngModel)]="groupBy">
      <option *ngFor="let item of ['Account', 'AccountNumber', 'Balance', 'Balance Pool', 'ClosingAvlBal', 'ClosingAvlBalCur', 'ClosingAvlBalDate', 'ClosingBalCur']">{{ item }}</option>
    </select>

    <label>Aggregation:</label>
    <select [(ngModel)]="aggregation">
      <option>Count</option>
      <option>Sum</option>
      <option>None</option>
    </select>

    <label>Aggregation Field:</label>
    <select [(ngModel)]="aggregationField">
      <option *ngFor="let item of ['Balance', 'ClosingAvlBal', 'ClosingBalance', 'Eq Balance', 'Opening Balance', 'System Balance', 'UserNum1', 'UserNum2']">{{ item }}</option>
    </select>

    <label>Column Field:</label>
    <select [(ngModel)]="columnField">
      <option *ngFor="let item of ['Account', 'AccountNumber', 'Balance', 'Balance Pool', 'ClosingAvlBal', 'ClosingAvlBalCur', 'ClosingAvlBalDate', 'ClosingBalCur']">{{ item }}</option>
    </select>

    <div class="template-section">
      <h4>Available Templates</h4>
      <button *ngFor="let template of availableTemplates" (click)="toggleTemplate(template)">
        {{ template }}
      </button>
      <div>
        <h4>Selected Templates</h4>
        <div *ngFor="let t of selectedTemplates">
          {{ t }} <button (click)="removeTemplate(t)">-</button>
        </div>
      </div>
    </div>
  </div>

  <div class="step-content" *ngIf="currentStep === 2">
    <p><strong>Name:</strong> {{ formData.name }}</p>
    <p><strong>Description:</strong> {{ formData.description }}</p>
    <p><strong>Visibility:</strong> {{ formData.isPublic ? 'Public' : 'Private' }}</p>
    <p><strong>Model:</strong> {{ model }}</p>
    <p><strong>GroupBy:</strong> {{ groupBy }}</p>
    <p><strong>Aggregation:</strong> {{ aggregation }}</p>
    <p><strong>Aggregation Field:</strong> {{ aggregationField }}</p>
    <p><strong>Column Field:</strong> {{ columnField }}</p>
    <p><strong>Templates:</strong> {{ selectedTemplates.join(', ') }}</p>
  </div>

  <div class="modal-footer">
    <button (click)="prevStep()" [disabled]="currentStep === 0">Previous</button>
    <button *ngIf="currentStep < steps.length - 1" (click)="nextStep()">Next</button>
    <button *ngIf="currentStep === steps.length - 1" (click)="submitDashboard()">Submit</button>
  </div>
</div>

css
.modal {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.2);
  padding: 20px;
  max-width: 800px;
  margin: 2rem auto;
}
.fullscreen {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
}
.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.steps {
  margin-top: 1rem;
}
.steps button {
  margin-right: 8px;
  padding: 6px 12px;
}
.steps button.active {
  background-color: #007bff;
  color: white;
}
.step-content {
  margin-top: 1rem;
}
.template-section {
  margin-top: 1rem;
}
.modal-footer {
  margin-top: 1.5rem;
  display: flex;
  justify-content: space-between;
}


manage dashboard comp html 
<!-- manage-dashboard.component.html --><div class="dashboard-table">
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Description</th>
        <th>Created By</th>
        <th>Type</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let dashboard of dashboards">
        <td>{{ dashboard.name }}</td>
        <td>{{ dashboard.description }}</td>
        <td>{{ dashboard.createdBy }}</td>
        <td>
          <span *ngIf="isInteractiveDashboard(dashboard)" class="interactive-badge">Interactive</span>
          <span *ngIf="!isInteractiveDashboard(dashboard)" class="blank-badge">Blank</span>
        </td>
        <td>
          <button (click)="editDashboard(dashboard)">✏️</button>
          <button (click)="deleteDashboard(dashboard.id)">🗑️</button>
        </td>
      </tr>
    </tbody>
  </table>
</div><!-- Modals --><app-blank-dashboard *ngIf="showBlankModal" [editData]="selectedBlankDashboard" (dashboardClose)="handleCloseBlank()"> </app-blank-dashboard>

<app-interactive-dashboard *ngIf="showInteractiveModal" [editData]="selectedInteractiveDashboard" (dashboardClose)="handleCloseInteractive()"> </app-interactive-dashboard>

ts

dashboards: DashboardDTO[] = [];
showBlankModal = false;
showInteractiveModal = false;
selectedBlankDashboard?: DashboardDTO;
selectedInteractiveDashboard?: DashboardDTO;

ngOnInit() {
  this.dashboardService.getAllDashboards().subscribe(data => {
    this.dashboards = data;
  });
}

isInteractiveDashboard(d: DashboardDTO): boolean {
  return !!(d.model && d.groupBy && d.aggregation); // Basic check for interactive fields
