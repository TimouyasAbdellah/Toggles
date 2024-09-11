Créer une application de gestion des toggles avec Angular pour le frontend et Java Spring Boot pour le backend implique plusieurs étapes. Je vais te donner un aperçu du projet avec du code pour chaque composant.

### Architecture générale
1. **Frontend** (Angular): Interface utilisateur pour gérer les toggles.
2. **Backend** (Spring Boot): API pour gérer les toggles côté serveur (ajouter, supprimer, modifier, consulter).
3. **Base de données**: PostgreSQL ou H2 pour stocker les informations des toggles.

#### 1. Backend (Spring Boot)

##### 1.1. Configuration du projet Spring Boot
1. Utilise **Spring Initializr** pour créer ton projet Spring Boot en choisissant les dépendances suivantes :
    - Spring Web
    - Spring Data JPA
    - H2 (ou PostgreSQL si tu veux)
    - Spring Boot DevTools (optionnel)
    - Lombok (optionnel, pour réduire le boilerplate)

##### 1.2. `application.properties` ou `application.yml`
Si tu utilises H2 comme base de données en mémoire, ton fichier `application.properties` ressemblerait à ceci :

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update
```

##### 1.3. Créer l'entité Toggle

```java
package com.example.toggleapp.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Toggle {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private boolean enabled;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
}
```

##### 1.4. Créer un repository Toggle

```java
package com.example.toggleapp.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.example.toggleapp.model.Toggle;

public interface ToggleRepository extends JpaRepository<Toggle, Long> {
}
```

##### 1.5. Créer le service Toggle

```java
package com.example.toggleapp.service;

import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.example.toggleapp.model.Toggle;
import com.example.toggleapp.repository.ToggleRepository;

@Service
public class ToggleService {

    @Autowired
    private ToggleRepository toggleRepository;

    public List<Toggle> getAllToggles() {
        return toggleRepository.findAll();
    }

    public Optional<Toggle> getToggleById(Long id) {
        return toggleRepository.findById(id);
    }

    public Toggle saveToggle(Toggle toggle) {
        return toggleRepository.save(toggle);
    }

    public void deleteToggle(Long id) {
        toggleRepository.deleteById(id);
    }
}
```

##### 1.6. Créer le contrôleur Toggle

```java
package com.example.toggleapp.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import java.util.Optional;

import com.example.toggleapp.model.Toggle;
import com.example.toggleapp.service.ToggleService;

@RestController
@RequestMapping("/api/toggles")
@CrossOrigin(origins = "http://localhost:4200")  // Pour permettre les requêtes CORS depuis Angular
public class ToggleController {

    @Autowired
    private ToggleService toggleService;

    @GetMapping
    public List<Toggle> getAllToggles() {
        return toggleService.getAllToggles();
    }

    @GetMapping("/{id}")
    public Optional<Toggle> getToggleById(@PathVariable Long id) {
        return toggleService.getToggleById(id);
    }

    @PostMapping
    public Toggle createToggle(@RequestBody Toggle toggle) {
        return toggleService.saveToggle(toggle);
    }

    @PutMapping("/{id}")
    public Toggle updateToggle(@PathVariable Long id, @RequestBody Toggle toggle) {
        toggle.setId(id);
        return toggleService.saveToggle(toggle);
    }

    @DeleteMapping("/{id}")
    public void deleteToggle(@PathVariable Long id) {
        toggleService.deleteToggle(id);
    }
}
```

#### 2. Frontend (Angular)

##### 2.1. Créer un projet Angular
Si tu n'as pas encore un projet Angular, crée-le avec la commande suivante :

```bash
ng new toggle-app
```

##### 2.2. Service Angular pour les requêtes HTTP

Crée un service Angular pour interagir avec l'API Spring Boot.

```bash
ng generate service services/toggle
```

Voici le code du service :

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Toggle } from '../models/toggle';

@Injectable({
  providedIn: 'root'
})
export class ToggleService {

  private apiUrl = 'http://localhost:8080/api/toggles';

  constructor(private http: HttpClient) { }

  getToggles(): Observable<Toggle[]> {
    return this.http.get<Toggle[]>(this.apiUrl);
  }

  getToggleById(id: number): Observable<Toggle> {
    return this.http.get<Toggle>(`${this.apiUrl}/${id}`);
  }

  createToggle(toggle: Toggle): Observable<Toggle> {
    return this.http.post<Toggle>(this.apiUrl, toggle);
  }

  updateToggle(id: number, toggle: Toggle): Observable<Toggle> {
    return this.http.put<Toggle>(`${this.apiUrl}/${id}`, toggle);
  }

  deleteToggle(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

##### 2.3. Modèle Toggle

Crée un modèle pour représenter les toggles.

```bash
ng generate class models/toggle
```

```typescript
export class Toggle {
  id: number;
  name: string;
  enabled: boolean;
}
```

##### 2.4. Composant de gestion des toggles

Crée un composant pour afficher et gérer les toggles.

```bash
ng generate component components/toggle-list
```

Le fichier `toggle-list.component.ts` pourrait ressembler à ceci :

```typescript
import { Component, OnInit } from '@angular/core';
import { ToggleService } from '../../services/toggle.service';
import { Toggle } from '../../models/toggle';

@Component({
  selector: 'app-toggle-list',
  templateUrl: './toggle-list.component.html',
  styleUrls: ['./toggle-list.component.css']
})
export class ToggleListComponent implements OnInit {

  toggles: Toggle[] = [];

  constructor(private toggleService: ToggleService) { }

  ngOnInit(): void {
    this.getToggles();
  }

  getToggles(): void {
    this.toggleService.getToggles().subscribe((data: Toggle[]) => {
      this.toggles = data;
    });
  }

  toggleStatus(toggle: Toggle): void {
    toggle.enabled = !toggle.enabled;
    this.toggleService.updateToggle(toggle.id, toggle).subscribe();
  }

  deleteToggle(id: number): void {
    this.toggleService.deleteToggle(id).subscribe(() => {
      this.getToggles();
    });
  }
}
```

##### 2.5. Template HTML pour `toggle-list.component.html`

```html
<div>
  <h1>List of Toggles</h1>
  <ul>
    <li *ngFor="let toggle of toggles">
      <span>{{ toggle.name }} ({{ toggle.enabled ? 'Enabled' : 'Disabled' }})</span>
      <button (click)="toggleStatus(toggle)">Toggle</button>
      <button (click)="deleteToggle(toggle.id)">Delete</button>
    </li>
  </ul>
</div>
```

#### 3. Lancer l'application

1. **Backend**: Démarre le serveur Spring Boot avec `mvn spring-boot:run`.
2. **Frontend**: Démarre Angular avec `ng serve`.
3. Accède à ton frontend via `http://localhost:4200`, et assure-toi que le backend fonctionne sur `http://localhost:8080`.

### Conclusion

Cette application simple de gestion des toggles te permet de créer, mettre à jour, supprimer et basculer les états des toggles. Tu peux bien sûr enrichir cela avec plus de fonctionnalités selon tes besoins (authentification, validation, etc.).
