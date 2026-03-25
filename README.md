#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "library.dat"

typedef struct {
    int id;
    char title[100];
    char author[50];
    int year;
} Book;

// Function prototypes
void addBook();
void displayBooks();
void searchBook();
void deleteBook();
void updateBook();
void countBooks();
void listByAuthor();
void listByYear();
void menu();

int main() {
    int choice;
    while (1) {
        menu();
        printf("\nEnter your choice: ");
        scanf("%d", &choice);
        getchar(); // consume newline

        switch (choice) {
            case 1: addBook(); break;
            case 2: displayBooks(); break;
            case 3: searchBook(); break;
            case 4: deleteBook(); break;
            case 5: updateBook(); break;
            case 6: countBooks(); break;
            case 7: listByAuthor(); break;
            case 8: listByYear(); break;
            case 9: printf("\nExiting program... Goodbye!\n"); exit(0);
            default: printf("\nInvalid choice! Try again.\n");
        }
    }
    return 0;
}

void menu() {
    printf("\n========== MINI LIBRARY CATALOGUE ==========\n");
    printf("1. Add a New Book\n");
    printf("2. Display All Books\n");
    printf("3. Search Book by ID\n");
    printf("4. Delete Book by ID\n");
    printf("5. Update Book Details\n");
    printf("6. Count Total Books\n");
    printf("7. List Books by Author\n");
    printf("8. List Books Published After Year\n");
    printf("9. Exit\n");
    printf("============================================\n");
}

void addBook() {
    FILE *fp = fopen(FILENAME, "ab");
    if (!fp) {
        printf("Error opening file!\n");
        return;
    }

    Book b;
    printf("Enter book ID: ");
    scanf("%d", &b.id);
    getchar();
    printf("Enter book title: ");
    fgets(b.title, 100, stdin);
    b.title[strcspn(b.title, "\n")] = '\0';
    printf("Enter author name: ");
    fgets(b.author, 50, stdin);
    b.author[strcspn(b.author, "\n")] = '\0';
    printf("Enter publication year: ");
    scanf("%d", &b.year);

    fwrite(&b, sizeof(Book), 1, fp);
    fclose(fp);

    printf("\nBook added successfully!\n");
}

void displayBooks() {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) {
        printf("No books found!\n");
        return;
    }

    Book b;
    printf("\n%-5s %-25s %-20s %-5s\n", "ID", "Title", "Author", "Year");
    printf("----------------------------------------------------------\n");
    while (fread(&b, sizeof(Book), 1, fp))
        printf("%-5d %-25s %-20s %-5d\n", b.id, b.title, b.author, b.year);

    fclose(fp);
}

void searchBook() {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) {
        printf("No books found!\n");
        return;
    }

    int id, found = 0;
    printf("Enter book ID to search: ");
    scanf("%d", &id);
    Book b;

    while (fread(&b, sizeof(Book), 1, fp)) {
        if (b.id == id) {
            printf("\nBook Found:\n");
            printf("ID: %d\nTitle: %s\nAuthor: %s\nYear: %d\n", b.id, b.title, b.author, b.year);
            found = 1;
            break;
        }
    }

    if (!found) printf("Book not found!\n");
    fclose(fp);
}

void deleteBook() {
    FILE *fp = fopen(FILENAME, "rb");
    FILE *temp = fopen("temp.dat", "wb");
    if (!fp || !temp) {
        printf("Error opening file!\n");
        return;
    }

    int id, found = 0;
    printf("Enter book ID to delete: ");
    scanf("%d", &id);
    Book b;

    while (fread(&b, sizeof(Book), 1, fp)) {
        if (b.id != id)
            fwrite(&b, sizeof(Book), 1, temp);
        else
            found = 1;
    }

    fclose(fp);
    fclose(temp);

    remove(FILENAME);
    rename("temp.dat", FILENAME);

    if (found)
        printf("Book deleted successfully!\n");
    else
        printf("Book not found!\n");
}

void updateBook() {
    FILE *fp = fopen(FILENAME, "r+b");
    if (!fp) {
        printf("Error opening file!\n");
        return;
    }

    int id, found = 0;
    printf("Enter book ID to update: ");
    scanf("%d", &id);
    getchar();

    Book b;
    while (fread(&b, sizeof(Book), 1, fp)) {
        if (b.id == id) {
            printf("Enter new title: ");
            fgets(b.title, 100, stdin);
            b.title[strcspn(b.title, "\n")] = '\0';
            printf("Enter new author: ");
            fgets(b.author, 50, stdin);
            b.author[strcspn(b.author, "\n")] = '\0';
            printf("Enter new year: ");
            scanf("%d", &b.year);

            fseek(fp, -sizeof(Book), SEEK_CUR);
            fwrite(&b, sizeof(Book), 1, fp);
            found = 1;
            break;
        }
    }

    fclose(fp);
    if (found)
        printf("Book updated successfully!\n");
    else
        printf("Book not found!\n");
}

void countBooks() {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) {
        printf("No books found!\n");
        return;
    }

    Book b;
    int count = 0;
    while (fread(&b, sizeof(Book), 1, fp)) count++;
    fclose(fp);
    printf("Total number of books: %d\n", count);
}

void listByAuthor() {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) {
        printf("No books found!\n");
        return;
    }

    char author[50];
    printf("Enter author name: ");
    getchar();
    fgets(author, 50, stdin);
    author[strcspn(author, "\n")] = '\0';

    Book b;
    int found = 0;
    printf("\nBooks by %s:\n", author);
    while (fread(&b, sizeof(Book), 1, fp)) {
        if (strcmp(b.author, author) == 0) {
            printf("ID: %d | Title: %s | Year: %d\n", b.id, b.title, b.year);
            found = 1;
        }
    }
    if (!found) printf("No books by this author.\n");
    fclose(fp);
}

void listByYear() {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) {
        printf("No books found!\n");
        return;
    }

    int year;
    printf("Enter the year: ");
    scanf("%d", &year);
    Book b;
    int found = 0;
    printf("\nBooks published after %d:\n", year);
    while (fread(&b, sizeof(Book), 1, fp)) {
        if (b.year > year) {
            printf("ID: %d | Title: %s | Author: %s | Year: %d\n", b.id, b.title, b.author, b.year);
            found = 1;
        }
    }
    if (!found) printf("No books found.\n");
    fclose(fp);
}
