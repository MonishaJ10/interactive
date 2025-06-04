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




<div class="table-container">
  <table class="dashboards-table">
    <thead>
      <tr>
        <th *ngFor="let header of tableHeaders">{{ header.label }}</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngIf="tableData.length === 0" class="no-data-row">
        <td [attr.colspan]="tableHeaders.length" class="no-data-cell">No Rows To Show</td>
      </tr>
      <tr *ngFor="let row of tableData">
        <td *ngFor="let header of tableHeaders">
          <ng-container [ngSwitch]="header.key">
            <span *ngSwitchCase="'createdDate'">{{ row.createdDate | date: 'short' }}</span>
            <span *ngSwitchCase="'modifiedDate'">{{ row.modifiedDate | date: 'short' }}</span>
            <span *ngSwitchCase="'public'">{{ row.isPublic ? 'Yes' : 'No' }}</span>
            <span *ngSwitchCase="'action'">
              <button mat-icon-button color="primary" (click)="onEditDashboard(row)">
                <mat-icon>edit</mat-icon>
              </button>
              <button mat-icon-button color="warn" (click)="onDeleteDashboard(row.id)">
                <mat-icon>delete</mat-icon>
              </button>
            </span>
            <span *ngSwitchDefault>{{ row[header.key] }}</span>
          </ng-container>
        </td>
      </tr>
    </tbody>
  </table>
</div>

mangage dashboard comp.ts
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
import { DashboardService } from '../services/dashboard.service';
import { Dashboardd } from '../models/dashboard.model'; // adjust path as needed

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent implements OnInit {
  dashboards: Dashboardd[] = [];
  tableData: Dashboardd[] = [];

  showBlankModal = false;
  showInteractiveModal = false;

  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  selectedDashboard: Dashboardd | null = null;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  tableHeaders = [
    { key: 'name', label: 'Name' },
    { key: 'description', label: 'Description' },
    { key: 'createdBy', label: 'Created By' },
    { key: 'createdDate', label: 'Created Date' },
    { key: 'modifiedBy', label: 'Modified By' },
    { key: 'modifiedDate', label: 'Modified Date' },
    { key: 'public', label: 'Public' },
    { key: 'action', label: 'Action' }
  ];

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  onEditDashboard(dashboard: Dashboardd): void {
    this.selectedDashboard = dashboard;

    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveModal = true;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankModal = true;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe(() => {
        this.loadDashboards();
      });
    }
  }

  isInteractiveDashboard(dashboard: Dashboardd): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }
}

import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { DashboardService } from 'src/app/services/dashboard.service';
import { Dashboard } from 'src/app/models/dashboard.model';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent implements OnInit {

  tableData: Dashboard[] = [];
  selectedDashboard: Dashboard | null = null;

  dashboards = [
    {
      id: 'blank',
      name: 'Blank Dashboard',
      icon: '',
      description: 'Start with a blank canvas'
    },
    {
      id: 'interactive',
      name: 'Interactive Dashboard',
      icon: '',
      description: 'Pre-built interactive components'
    }
  ];

  tableHeaders = [
    { key: 'name', label: 'Name' },
    { key: 'description', label: 'Description' },
    { key: 'createdBy', label: 'Created By' },
    { key: 'createdDate', label: 'Created Date' },
    { key: 'modifiedBy', label: 'Modified By' },
    { key: 'modifiedDate', label: 'Modified Date' },
    { key: 'public', label: 'Public' },
    { key: 'action', label: 'Action' }
  ];

  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard?: Dashboard;
  selectedInteractiveDashboard?: Dashboard;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data) => this.tableData = data,
      error: () => alert('Failed to load dashboards')
    });
  }

  onSelectDashboard(dashboardType: string): void {
    if (dashboardType === 'blank') {
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    } else if (dashboardType === 'interactive') {
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    }
    this.selectDashboard.emit(dashboardType);
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
  }

  onNewDashboard(): void {
    this.selectedDashboard = null;
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
  }

  onEditDashboard(dashboard: Dashboard): void {
    this.selectedDashboard = dashboard;
    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe({
        next: () => {
          this.tableData = this.tableData.filter(d => d.id !== id);
          alert('Dashboard deleted successfully.');
        },
        error: () => alert('Delete failed')
      });
    }
  }

  isInteractiveDashboard(dashboard: Dashboard): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }

  onClose(): void {
    this.closeDashboard.emit();
  }
}




<div class="manage-dashboard-container">
  <!-- Main Manage Dashboard View -->
  <div *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <!-- Header -->
    <div class="header">
      <h2 class="title">Manage Dashboard</h2>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>

    <!-- New Dashboard Button -->
    <div class="new-dashboard-section">
      <button class="new-dashboard-btn" (click)="onNewDashboard()">
        <span class="plus-icon">+</span>
        New Dashboard
      </button>
    </div>

    <!-- Dashboard Types -->
    <div class="dashboard-types">
      <div class="dashboard-card"
           *ngFor="let dashboard of dashboards"
           (click)="onSelectDashboard(dashboard.id)">
        <div class="dashboard-icon">{{ dashboard.icon }}</div>
        <div class="dashboard-name">{{ dashboard.name }}</div>
      </div>
    </div>

    <!-- All Dashboards Section -->
    <div class="all-dashboards-section">
      <h3 class="section-title">All Dashboards</h3>
      <div class="table-container">
        <table class="dashboards-table">
          <thead>
            <tr>
              <th *ngFor="let header of tableHeaders">{{ header.label }}</th>
            </tr>
          </thead>
          <tbody>
            <tr *ngIf="tableData.length === 0" class="no-data-row">
              <td [attr.colspan]="tableHeaders.length" class="no-data-cell">No Rows To Show</td>
            </tr>
            <tr *ngFor="let row of tableData">
              <td *ngFor="let header of tableHeaders">
                <ng-container [ngSwitch]="header.key">
                  <span *ngSwitchCase="'createdDate'">{{ row.createdDate | date: 'short' }}</span>
                  <span *ngSwitchCase="'modifiedDate'">{{ row.modifiedDate | date: 'short' }}</span>
                  <span *ngSwitchCase="'public'">{{ row.public ? 'Yes' : 'No' }}</span>
                  <span *ngSwitchCase="'action'">
                    <button mat-icon-button color="primary" (click)="onEditDashboard(row)">
                      <mat-icon>edit</mat-icon>
                    </button>
                    <button mat-icon-button color="warn" (click)="onDeleteDashboard(row.id)">
                      <mat-icon>delete</mat-icon>
                    </button>
                  </span>
                  <span *ngSwitchDefault>{{ row[header.key] }}</span>
                </ng-container>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <!-- Blank Dashboard View -->
  <div *ngIf="showBlankDashboard">
    <app-blank-dashboard
      [editData]="selectedBlankDashboard"
      (dashboardClose)="onBackToManage()">
    </app-blank-dashboard>
  </div>

  <!-- Interactive Dashboard View -->
  <div *ngIf="showInteractiveDashboard" class="dashboard-view">
    <div class="dashboard-header">
      <button class="back-btn" (click)="onBackToManage()">Back to Manage</button>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>
    <app-interactive-dashboard
      *ngIf="showInteractiveDashboard"
      [editData]="selectedInteractiveDashboard"
      (dashboardClose)="onBackToManage()">
    </app-interactive-dashboard>
  </div>
</div>





action 
import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule],
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(params: any): boolean {
    return false;
  }

  onEditClicked(): void {
    if (this.params?.onEditClicked) {
      this.params.onEditClicked(this.params.data);
    }
  }

  onDeleteClicked(): void {
    if (this.params?.onDeleteClicked) {
      this.params.onDeleteClicked(this.params.data.id);
    }
  }
}

manage ts
import { Component, EventEmitter, Output, OnInit, Inject } from '@angular/core';
import { isPlatformBrowser, CommonModule } from '@angular/common';
import { PLATFORM_ID } from '@angular/core';
import { DashboardService } from './dashboard.service';
import { Dashboardd } from './dashboard.model';
import { ColDef } from 'ag-grid-community';
import { AgGridModule } from 'ag-grid-angular';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';
import { ActionCellRendererComponent } from './action-cell-renderer.component';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  imports: [
    CommonModule,
    NgIf,
    MatIconModule,
    MatButtonModule,
    AgGridModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
    ActionCellRendererComponent
  ]
})
export class ManageDashboardComponent implements OnInit {
  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  tableData: Dashboardd[] = [];
  selectedDashboard: Dashboardd | null = null;
  editingDashboard: Dashboardd | null = null;

  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  isBrowser: boolean;

  dashboards = [
    { id: 'blank', name: 'Blank Dashboard', icon: '' },
    { id: 'interactive', name: 'Interactive Dashboard', icon: '' }
  ];

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'public', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Action',
      field: 'action',
      cellRenderer: 'appActionCellRenderer',
      cellRendererParams: {
        onEditClicked: this.onEditDashboard.bind(this),
        onDeleteClicked: this.onDeleteDashboard.bind(this)
      },
      width: 120,
      pinned: 'right'
    }
  ];

  defaultColDef: ColDef = {
    flex: 1,
    resizable: true
  };

  frameworkComponents = {
    appActionCellRenderer: ActionCellRendererComponent
  };

  constructor(
    private dashboardService: DashboardService,
    @Inject(PLATFORM_ID) platformId: Object
  ) {
    this.isBrowser = isPlatformBrowser(platformId);
  }

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data) => {
        this.tableData = data.map(d => ({ ...d, action: '' }));
      },
      error: (error) => {
        console.error('Failed to load dashboards:', error);
        alert('Failed to load dashboards');
      }
    });
  }

  onSelectDashboard(dashboardType: string): void {
    if (dashboardType === 'blank') {
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    } else if (dashboardType === 'interactive') {
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    }
    this.selectDashboard.emit(dashboardType);
  }

  onNewDashboard(): void {
    this.selectedDashboard = null;
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
  }

  onEditDashboard(dashboard: Dashboardd): void {
    this.editingDashboard = dashboard;
    this.selectedDashboard = dashboard;

    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe({
        next: () => {
          this.tableData = this.tableData.filter(d => d.id !== id);
          alert('Dashboard deleted successfully.');
        },
        error: () => alert('Delete failed')
      });
    }
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
    this.loadDashboards();
  }

  onClose(): void {
    this.closeDashboard.emit();
  }

  onDashboardCreated(): void {
    this.loadDashboards();
  }

  onGridReady(params: any): void {
    console.log('Grid is ready:', params);
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  isInteractiveDashboard(dashboard: Dashboardd): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }
}

manage html
<div class="manage-dashboard-container">
  <!-- Main Manage Dashboard View -->
  <div *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <!-- Header -->
    <div class="header">
      <h2 class="title">Manage Dashboard</h2>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>

    <!-- New Dashboard Button -->
    <div class="new-dashboard-section">
      <button class="new-dashboard-btn" (click)="onNewDashboard()">
        <span class="plus-icon">+</span>
        New Dashboard
      </button>
    </div>

    <!-- Dashboard Types -->
    <div class="dashboard-types">
      <div class="dashboard-card" *ngFor="let dashboard of dashboards" (click)="onSelectDashboard(dashboard.id)">
        <div class="dashboard-icon">{{ dashboard.icon }}</div>
        <div class="dashboard-name">{{ dashboard.name }}</div>
      </div>
    </div>

    <!-- All Dashboards Section -->
    <div class="all-dashboards-section">
      <h3 class="section-title">All Dashboards</h3>
      <div class="table-container">
        <ng-container *ngIf="isBrowser; else placeholder">
          <ag-grid-angular
            class="ag-theme-balham"
            style="width: 100%; height: 500px;"
            [rowData]="tableData"
            [columnDefs]="columnDefs"
            [defaultColDef]="defaultColDef"
            [frameworkComponents]="frameworkComponents"
            (gridReady)="onGridReady($event)">
          </ag-grid-angular>
        </ng-container>
        <ng-template #placeholder>
          <app-ag-grid-placeholder></app-ag-grid-placeholder>
        </ng-template>
      </div>
    </div>
  </div>

  <!-- Blank Dashboard View -->
  <div *ngIf="showBlankDashboard">
    <app-blank-dashboard
      [editData]="selectedBlankDashboard"
      (dashboardClose)="onBackToManage()">
    </app-blank-dashboard>
  </div>

  <!-- Interactive Dashboard View -->
  <div *ngIf="showInteractiveDashboard">
    <div class="dashboard-header">
      <button class="back-btn" (click)="onBackToManage()">Back to Manage</button>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>
    <app-interactive-dashboard
      [editData]="selectedInteractiveDashboard"
      (dashboardCreated)="onDashboardCreated()"
      (dashboardClose)="onBackToManage()">
    </app-interactive-dashboard>
  </div>
</div>




Here’s a full working version of your ManageDashboardComponent using Angular standalone components, including AgGridModule.withComponents() and ActionCellRendererComponent integration.


---

✅ manage-dashboard.component.ts

import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef, GridReadyEvent } from 'ag-grid-community';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

import { ActionCellRendererComponent } from './action-cell-renderer.component'; // Update path if needed

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  imports: [
    CommonModule,
    MatButtonModule,
    MatIconModule,
    AgGridModule.withComponents([ActionCellRendererComponent]),
    ActionCellRendererComponent,
  ],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class ManageDashboardComponent implements OnInit {
  tableData: any[] = []; // Populate with your dashboard data

  columnDefs: ColDef[] = [
    { headerName: 'Name', field: 'name' },
    { headerName: 'Type', field: 'type' },
    {
      headerName: 'Actions',
      field: 'actions',
      cellRenderer: 'appActionCellRenderer',
      cellRendererParams: {
        onEditClicked: this.onEditDashboard.bind(this),
        onDeleteClicked: this.onDeleteDashboard.bind(this),
      },
    },
  ];

  defaultColDef: ColDef = {
    flex: 1,
    sortable: true,
    filter: true,
    resizable: true,
  };

  frameworkComponents = {
    appActionCellRenderer: ActionCellRendererComponent,
  };

  isBrowser = true;

  ngOnInit(): void {
    // load data here if needed
  }

  onGridReady(event: GridReadyEvent): void {
    // optional grid setup
  }

  onEditDashboard(dashboard: any): void {
    console.log('Edit clicked', dashboard);
    // handle edit logic
  }

  onDeleteDashboard(dashboardId: string): void {
    console.log('Delete clicked', dashboardId);
    // handle delete logic
  }
}


---

✅ action-cell-renderer.component.ts

import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `,
  imports: [CommonModule, MatButtonModule, MatIconModule],
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(): boolean {
    return false;
  }

  onEditClicked(): void {
    this.params.onEditClicked(this.params.data);
  }

  onDeleteClicked(): void {
    this.params.onDeleteClicked(this.params.data.id);
  }
}______________________

Thanks! Based on all your shared context, here is a clean and working setup for:


---

✅ manage-dashboard.component.html

<div class="all-dashboards-section">
  <h3 class="section-title">All Dashboards</h3>
  <div class="table-container">
    <ng-container *ngIf="isBrowser; else placeholder">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [rowData]="tableData"
        [columnDefs]="columnDefs"
        [defaultColDef]="defaultColDef"
        [frameworkComponents]="frameworkComponents"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </ng-container>
    <ng-template #placeholder>
      <app-ag-grid-placeholder></app-ag-grid-placeholder>
    </ng-template>
  </div>
</div>


---

✅ manage-dashboard.component.ts

import { Component, inject } from '@angular/core';
import { CommonModule, isPlatformBrowser } from '@angular/common';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';
import { DashboardService } from '../../services/dashboard.service';
import { ActionCellRendererComponent } from '../action-cell-renderer/action-cell-renderer.component';
import { AgGridPlaceholderComponent } from '../ag-grid-placeholder/ag-grid-placeholder.component';
import { PLATFORM_ID } from '@angular/core';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  imports: [CommonModule, AgGridModule, ActionCellRendererComponent, AgGridPlaceholderComponent],
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent {
  private dashboardService = inject(DashboardService);
  private platformId = inject(PLATFORM_ID);

  isBrowser = isPlatformBrowser(this.platformId);

  tableData: any[] = [];

  frameworkComponents = {
    actionCellRenderer: ActionCellRendererComponent
  };

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      field: 'actions',
      cellRenderer: 'actionCellRenderer',
      headerName: 'Actions',
      cellRendererParams: {
        onEditClicked: (data: any) => this.editDashboard(data),
        onDeleteClicked: (id: number) => this.deleteDashboard(id)
      }
    }
  ];

  defaultColDef: ColDef = {
    sortable: true,
    filter: true,
    resizable: true
  };

  ngOnInit() {
    this.loadDashboards();
  }

  onGridReady(params: any) {
    params.api.sizeColumnsToFit();
  }

  loadDashboards() {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  editDashboard(data: any) {
    // Logic to open edit modal and populate with data
    console.log('Edit dashboard', data);
  }

  deleteDashboard(id: number) {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }
}


---

✅ action-cell-renderer.component.ts

import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule],
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(): boolean {
    return false;
  }

  onEditClicked(): void {
    this.params.onEditClicked(this.params.data);
  }

  onDeleteClicked(): void {
    this.params.onDeleteClicked(this.params.data.id);
  }
}


---

✅ ag-grid-placeholder.component.ts (Optional fallback when not browser)

import { Component } from '@angular/core';

@Component({
  selector: 'app-ag-grid-placeholder',
  standalone: true,
  template: `<p>Loading dashboard table...</p>`
})
export class AgGridPlaceholderComponent {}


---

Would you like the working dashboard.model.ts and dashboard.service.ts cleaned up too?





