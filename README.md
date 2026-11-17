# DEVELOPING-AN-ANDROID-APPLICATION-FOR-ADDRESS-BOOK-

## AIM:
To develop an android application for address book.

## APPARATUS REQUIRED:
ØComputer
ØAndroid studio software


## THEORY:
The Address Book App is designed to store and display contact details such as a person's name and phone number. The user inputs the name and phone number into EditText fields, and by clicking the "Save Contact" button, the app displays the entered contact information in a TextView. This experiment introduces the basics of data handling in Android, including retrieving user input, storing it temporarily, and displaying it on the screen. It also demonstrates how to interact with basic UI components and use event listeners to respond to user actions. The Address Book App serves as a foundation for more complex applications involving data storage and manipulation.

## PROCEDURE:

1.Open Android Studio and then click on File -> New -> New project. 13. Then type the application name as “ex.no.2″ and click Next.
2.Then select the Minimum SDK as shown below and click next. 15. Then select the Empty Activity and click next.
3.Finally click Finish.
4.Click on app -> java -> com.example -> MainActivity.java
5.Now click on Text and type the program, so now the programming part of the main activity is completed.
6.Click on app -> res -> layout -> activity_main.xml.
7.Now click on Text and type the program, so now the designing part of Activity main is also completed.
8.Select the suitable available device to display the output. 22. Now run the application to see the output. 

## PROGRAM:

 MainActivity.kt

package com.example.addressbook;

import android.app.Activity;
import android.app.AlertDialog;
import android.os.Bundle;
import android.database.Cursor;
import android.graphics.Color;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import java.util.ArrayList;

public class MainActivity extends Activity {

    EditText nameInput, phoneInput, emailInput, addressInput;
    Button saveButton;
    ListView contactList;

    DatabaseHelper db;
    ArrayList<Integer> contactIds;
    ArrayList<String> contacts;
    ArrayAdapter<String> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        nameInput = findViewById(R.id.nameInput);
        phoneInput = findViewById(R.id.phoneInput);
        emailInput = findViewById(R.id.emailInput);
        addressInput = findViewById(R.id.addressInput);
        saveButton = findViewById(R.id.saveButton);
        contactList = findViewById(R.id.contactList);

        db = new DatabaseHelper(this);

        contacts = new ArrayList<>();
        contactIds = new ArrayList<>();

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                contacts
        );

        contactList.setAdapter(adapter);

        saveButton.setOnClickListener(v -> saveContact());

        contactList.setOnItemClickListener((parent, view,
                                            position, id) -> {
            editContact(position);
        });

        contactList.setOnItemLongClickListener(
                (parent, view, position, id) -> {
                    confirmDelete(position);
                    return true;
                });

        loadContacts();
    }

    private void saveContact() {
        String name = nameInput.getText().toString().trim();
        String phone = phoneInput.getText().toString().trim();
        String email = emailInput.getText().toString().trim();
        String address = addressInput.getText().toString().trim();

        if (name.isEmpty() || phone.isEmpty()) {
            Toast.makeText(this,
                    "Enter name and phone number",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        boolean result = db.addContact(
                name, phone, email, address);

        if (result) {
            Toast.makeText(this, "Contact saved",
                    Toast.LENGTH_SHORT).show();

            clearFields();
            loadContacts();
        } else {
            Toast.makeText(this, "Unable to save contact",
                    Toast.LENGTH_SHORT).show();
        }
    }

    private void loadContacts() {
        contacts.clear();
        contactIds.clear();

        try (Cursor cursor = db.getContacts()) {
            while (cursor.moveToNext()) {
                int id = cursor.getInt(
                        cursor.getColumnIndexOrThrow("id"));

                String name = cursor.getString(
                        cursor.getColumnIndexOrThrow("name"));

                String phone = cursor.getString(
                        cursor.getColumnIndexOrThrow("phone"));

                String email = cursor.getString(
                        cursor.getColumnIndexOrThrow("email"));

                String address = cursor.getString(
                        cursor.getColumnIndexOrThrow("address"));

                contactIds.add(id);

                contacts.add(name + "\nPhone: " + phone
                        + "\nEmail: " + email
                        + "\nAddress: " + address);
            }
        }

        adapter.notifyDataSetChanged();
    }

    private void clearFields() {
        nameInput.setText("");
        phoneInput.setText("");
        emailInput.setText("");
        addressInput.setText("");
        nameInput.requestFocus();
    }

    private void editContact(int position) {
        int id = contactIds.get(position);

        try (Cursor cursor = db.getContacts()) {
            while (cursor.moveToNext()) {
                if (cursor.getInt(
                        cursor.getColumnIndexOrThrow("id")) == id) {

                    String name = cursor.getString(
                            cursor.getColumnIndexOrThrow("name"));
                    String phone = cursor.getString(
                            cursor.getColumnIndexOrThrow("phone"));
                    String email = cursor.getString(
                            cursor.getColumnIndexOrThrow("email"));
                    String address = cursor.getString(
                            cursor.getColumnIndexOrThrow("address"));

                    showEditDialog(id, name, phone, email, address);
                    return;
                }
            }
        }
    }

    private void showEditDialog(int id, String name,
                                String phone, String email,
                                String address) {

        LinearLayout layout = new LinearLayout(this);
        layout.setOrientation(LinearLayout.VERTICAL);
        layout.setPadding(32, 8, 32, 0);

        EditText editName = makeField("Name", name);
        EditText editPhone = makeField("Phone", phone);
        EditText editEmail = makeField("Email", email);
        EditText editAddress = makeField("Address", address);

        layout.addView(editName);
        layout.addView(editPhone);
        layout.addView(editEmail);
        layout.addView(editAddress);

        AlertDialog dialog = new AlertDialog.Builder(this)
                .setTitle("Edit Contact")
                .setView(layout)
                .setPositiveButton("Update", null)
                .setNegativeButton("Cancel", null)
                .create();

        dialog.setOnShowListener(d -> {
            dialog.getButton(AlertDialog.BUTTON_POSITIVE)
                    .setOnClickListener(v -> {

                String newName = editName.getText()
                        .toString().trim();
                String newPhone = editPhone.getText()
                        .toString().trim();

                if (newName.isEmpty() || newPhone.isEmpty()) {
                    Toast.makeText(this,
                            "Name and phone are required",
                            Toast.LENGTH_SHORT).show();
                    return;
                }

                boolean updated = db.updateContact(
                        id,
                        newName,
                        newPhone,
                        editEmail.getText().toString().trim(),
                        editAddress.getText().toString().trim()
                );

                if (updated) {
                    loadContacts();
                    Toast.makeText(this, "Contact updated",
                            Toast.LENGTH_SHORT).show();
                    dialog.dismiss();
                } else {
                    Toast.makeText(this, "Update failed",
                            Toast.LENGTH_SHORT).show();
                }
            });
        });

        dialog.show();
    }

    private EditText makeField(String hint, String value) {
        EditText field = new EditText(this);
        field.setHint(hint);
        field.setText(value);
        field.setLayoutParams(new LinearLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT));
        return field;
    }

    private void confirmDelete(int position) {
        int id = contactIds.get(position);

        new AlertDialog.Builder(this)
                .setTitle("Delete Contact")
                .setMessage("Do you want to delete this contact?")
                .setPositiveButton("Delete", (dialog, which) -> {
                    if (db.deleteContact(id)) {
                        loadContacts();
                        Toast.makeText(this,
                                "Contact deleted",
                                Toast.LENGTH_SHORT).show();
                    } else {
                        Toast.makeText(this,
                                "Delete failed",
                                Toast.LENGTH_SHORT).show();
                    }
                })
                .setNegativeButton("Cancel", null)
                .show();
    }

    @Override
    protected void onDestroy() {
        db.close();
        super.onDestroy();
    }
}


## OUTPUT:

<img width="2076" height="2152" alt="Screenshot_20261028_224858" src="https://github.com/user-attachments/assets/81bc24a7-d27e-4520-8c33-75a4c704c3e4" />



## RESULT:
Thus, the Android app for storing and displaying contacts is developed and the output is verified.
