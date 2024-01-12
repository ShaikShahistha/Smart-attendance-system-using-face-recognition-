import cv2
import os

# Function to add a new student photo to the 'student_photos' folder
def add_student_photo():
    # Create the 'student_photos' folder if it doesn't exist
    photo_dir = 'student_photos'
    os.makedirs(photo_dir, exist_ok=True)

    # Open a connection to the camera (0 represents the default camera)
    cap = cv2.VideoCapture(0)

    # Take student's name as input
    student_name = input("Enter the student's name: ")

    while True:
        # Capture a single frame
        ret, frame = cap.read()

        # Display the frame
        cv2.imshow('Capture Student Photo', frame)

        # Wait for the user to press 'Space' to capture the photo
        key = cv2.waitKey(1) & 0xFF
        if key == 32:  # ASCII code for Space
            # Save the captured photo in the 'student_photos' folder
            photo_path = os.path.join(photo_dir, f'{student_name}.jpg')
            cv2.imwrite(photo_path, frame)
            print(f"Student {student_name}'s photo saved at {photo_path}")
            break
        elif key == 27:  # ASCII code for Esc
            # Break the loop if the user presses 'Esc'
            print("Photo capture canceled.")
            break

    # Release the camera and close the OpenCV window
    cap.release()
    cv2.destroyAllWindows()

# Main program
add_student_photo()



import cv2
import face_recognition
import os
import openpyxl
from openpyxl import Workbook
from datetime import datetime

def load_student_images(folder_path):
    student_images = {}
    for filename in os.listdir(folder_path):
        if filename.endswith(('.jpg', '.jpeg', '.png')):
            student_name = os.path.splitext(filename)[0]
            image_path = os.path.join(folder_path, filename)
            student_images[student_name] = face_recognition.load_image_file(image_path)
    return student_images

def mark_attendance(student_name, attendance_sheet):
    current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    attendance_sheet.append([current_time, student_name])
    print(f"{student_name} marked present at {current_time}")

def main():
    # Path to the folder containing student photos
    photo_folder_path = "student_photos"

    # Load student images from the folder
    student_images = load_student_images(photo_folder_path)

    # Create or load attendance Excel sheet
    excel_file_path = "attendance.xlsx"
    if os.path.exists(excel_file_path):
        workbook = openpyxl.load_workbook(excel_file_path)
    else:
        workbook = Workbook()

    # Use the default active sheet
    attendance_sheet = workbook.active

    # Load face encodings and names for each student
    student_encodings = {}
    for student_name, image in student_images.items():
        encoding = face_recognition.face_encodings(image)
        if len(encoding) > 0:
            student_encodings[student_name] = (encoding[0], False)

    # Open the camera (adjust the camera index as needed)
    cap = cv2.VideoCapture(0)

    while True:
        ret, frame = cap.read()
        if not ret:
            print("Failed to capture frame.")
            break

        # Find all face locations and face encodings in the current frame
        face_locations = face_recognition.face_locations(frame)
        face_encodings = face_recognition.face_encodings(frame, face_locations)

        # Check if any recognized face matches with student faces
        for (top, right, bottom, left), face_encoding in zip(face_locations, face_encodings):
            for student_name, (encoding, marked) in student_encodings.items():
                if not marked:
                    matches = face_recognition.compare_faces([encoding], face_encoding)
                    if any(matches):
                        mark_attendance(student_name, attendance_sheet)
                        student_encodings[student_name] = (encoding, True)

                    # Draw a frame around the face with the student's name
                    cv2.rectangle(frame, (left, top), (right, bottom), (0, 255, 0), 2)
                    font = cv2.FONT_HERSHEY_DUPLEX
                    cv2.putText(frame, student_name, (left + 6, bottom - 6), font, 0.5, (255, 255, 255), 1)

        # Display the frame
        cv2.imshow('Attendance System', frame)

        # Exit the loop if 'q' is pressed
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    # Release the camera and close the window
    cap.release()
    cv2.destroyAllWindows()

    # Save the updated attendance sheet
    workbook.save(excel_file_path)

if __name__ == "__main__":
    main()
