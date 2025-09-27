Step 1: Install Angular CLI

•	npm install -g @angular/cli
•	ng version
•	ng new Crud
•	cd Crud
•	ng serve

create this   Manually file crud.service.ts
// src/app/crud.service.ts
import { Injectable } from '@angular/core';

export interface User {
  id: number;
  name: string;
  state: string;
  contact: string;
  email: string;
  image?: string;
}

@Injectable({
  providedIn: 'root'
})
export class CrudService {
  private users: User[] = [];
  private nextId = 1;

  getUsers() { return this.users; }

  addUser(user: User) {
    user.id = this.nextId++;
    this.users.push(user);
  }

  updateUser(id: number, updatedUser: User) {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) this.users[index] = { ...updatedUser, id };
  }

  deleteUser(id: number) {
    this.users = this.users.filter(u => u.id !== id);
  }

  deleteImage(id: number) {
    const user = this.users.find(u => u.id === id);
    if (user) user.image = undefined;
  }
}
	app.component.ts – Component logic


import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { CrudService, User } from './crud.service';

@Component({
  selector: 'app-root',
  standalone: true,        // <-- important
  imports: [FormsModule, CommonModule], // <-- standalone imports
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  form: Partial<User> = {};
  users: User[] = [];
  editId: number | null = null;
  imagePreview: string | ArrayBuffer | null = null;

  constructor(private crudService: CrudService) {
    this.users = this.crudService.getUsers();
  }

  onFileChange(event: any) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = () => {
        this.imagePreview = reader.result;
        this.form.image = reader.result as string;
      };
      reader.readAsDataURL(file);
    }
  }

  onSubmit() {
    if (this.editId) {
      this.crudService.updateUser(this.editId, this.form as User);
      this.editId = null;
    } else {
      this.crudService.addUser(this.form as User);
    }
    this.form = {};
    this.imagePreview = null;
    this.users = this.crudService.getUsers();
  }

  onEdit(user: User) {
    this.form = { ...user };
    this.editId = user.id;
    this.imagePreview = user.image || null;
  }

  onDelete(id: number) {
    this.crudService.deleteUser(id);
    this.users = this.crudService.getUsers();
  }

  onDeleteImage(id: number) {
    this.crudService.deleteImage(id);
    this.users = this.crudService.getUsers();
  }
}




	app.component.html
<div class="container">
  <h2>Angular CRUD with Image Upload</h2>

  <form (ngSubmit)="onSubmit()" class="form">
    <input type="text" [(ngModel)]="form.name" name="name" placeholder="Name" required />
    <input type="text" [(ngModel)]="form.state" name="state" placeholder="State" required />
    <input type="text" [(ngModel)]="form.contact" name="contact" placeholder="Contact" required />
    <input type="email" [(ngModel)]="form.email" name="email" placeholder="Email" required />

    <input type="file" (change)="onFileChange($event)" />
    <div *ngIf="imagePreview">
      <img [src]="imagePreview" width="100" />
    </div>

    <button type="submit">{{ editId ? 'Update' : 'Add' }}</button>
  </form>

  <hr />

  <h3>User List</h3>
  <table border="1" cellpadding="5">
    <thead>
      <tr>
        <th>Name</th>
        <th>State</th>
        <th>Contact</th>
        <th>Email</th>
        <th>Image</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let user of users">
        <td>{{ user.name }}</td>
        <td>{{ user.state }}</td>
        <td>{{ user.contact }}</td>
        <td>{{ user.email }}</td>
        <td>
          <img *ngIf="user.image" [src]="user.image" width="80" />
        </td>
        <td>
          <button (click)="onEdit(user)">Edit</button>
          <button (click)="onDelete(user.id)">Delete</button>
          <button *ngIf="user.image" (click)="onDeleteImage(user.id)">Delete Image</button>
        </td>
      </tr>
    </tbody>
  </table>
</div>



	app.component.css – Styles

.container {
  padding: 20px;
}
.form {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  max-width: 300px;
}
.form input, .form button {
  margin: 5px 0;
  padding: 8px;
}
table {
  width: 100%;
  border-collapse: collapse;
}
------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------


import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';

export interface User {
  id: number;
  name: string;
  state: string;
  contact: string;
  email: string;
  image?: string;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [FormsModule, CommonModule],
  template: `
    <div class="container">
      <h2>Angular CRUD with Image Upload</h2>

      <form (ngSubmit)="onSubmit()" class="form">
        <input type="text" [(ngModel)]="form.name" name="name" placeholder="Name" required />
        <input type="text" [(ngModel)]="form.state" name="state" placeholder="State" required />
        <input type="text" [(ngModel)]="form.contact" name="contact" placeholder="Contact" required />
        <input type="email" [(ngModel)]="form.email" name="email" placeholder="Email" required />

        <input type="file" (change)="onFileChange($event)" />
        <div *ngIf="imagePreview">
          <img [src]="imagePreview" width="100" />
        </div>

        <button type="submit">{{ editId ? 'Update' : 'Add' }}</button>
      </form>

      <hr />

      <h3>User List</h3>
      <table border="1" cellpadding="5">
        <thead>
          <tr>
            <th>Name</th>
            <th>State</th>
            <th>Contact</th>
            <th>Email</th>
            <th>Image</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let user of users">
            <td>{{ user.name }}</td>
            <td>{{ user.state }}</td>
            <td>{{ user.contact }}</td>
            <td>{{ user.email }}</td>
            <td>
              <img *ngIf="user.image" [src]="user.image" width="80" />
            </td>
            <td>
              <button (click)="onEdit(user)">Edit</button>
              <button (click)="onDelete(user.id)">Delete</button>
              <button *ngIf="user.image" (click)="onDeleteImage(user.id)">Delete Image</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  `,
  styles: [`
    .container { padding: 20px; }
    .form { margin-bottom: 20px; display: flex; flex-direction: column; max-width: 300px; }
    .form input, .form button { margin: 5px 0; padding: 8px; }
    table { width: 100%; border-collapse: collapse; }
  `]
})
export class App {
  // Form and user list
  form: Partial<User> = {};
  users: User[] = [];
  editId: number | null = null;
  imagePreview: string | ArrayBuffer | null = null;

  // Simulate service inside component
  private nextId = 1;

  // File upload preview
  onFileChange(event: any) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = () => {
        this.imagePreview = reader.result;
        this.form.image = reader.result as string;
      };
      reader.readAsDataURL(file);
    }
  }

  // Add or update user
  onSubmit() {
    if (this.editId) {
      const index = this.users.findIndex(u => u.id === this.editId);
      if (index !== -1) this.users[index] = { ...this.form as User, id: this.editId };
      this.editId = null;
    } else {
      this.form.id = this.nextId++;
      this.users.push(this.form as User);
    }
    this.form = {};
    this.imagePreview = null;
  }

  // Edit user
  onEdit(user: User) {
    this.form = { ...user };
    this.editId = user.id;
    this.imagePreview = user.image || null;
  }

  // Delete user
  onDelete(id: number) {
    this.users = this.users.filter(u => u.id !== id);
  }

  // Delete user image
  onDeleteImage(id: number) {
    const user = this.users.find(u => u.id === id);
    if (user) user.image = undefined;
  }
}
