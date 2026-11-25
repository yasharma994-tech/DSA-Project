import java.util.*;

public class Main {

    // Student ki details store karne ke liye Helper Class
    static class Student {
        int id;
        String name;
        int rowIndex;

        Student(int id, String name, int rowIndex) {
            this.id = id;
            this.name = name;
            this.rowIndex = rowIndex;
        }
    }

    // Sorting ke liye Helper Class
    static class StudentReport {
        int id;
        String name;
        double percentage;

        StudentReport(int id, String name, double percentage) {
            this.id = id;
            this.name = name;
            this.percentage = percentage;
        }
    }

    static class AttendanceTracker {
        int numDays;
        int maxStudents;
        int currentStudents = 0;

        // 1. 2D Array: Rows = Students, Columns = Days (1 = Present, 0 = Absent)
        int[][] attendanceGrid;

        // 2. Hashing (HashMap): Student ID se O(1) instant lookup ke liye
        HashMap<Integer, Student> studentMap;

        public AttendanceTracker(int maxStudents, int numDays) {
            this.maxStudents = maxStudents;
            this.numDays = numDays;
            this.attendanceGrid = new int[maxStudents][numDays];
            this.studentMap = new HashMap<>();
        }

        // Student add karne ka method
        public void addStudent(int id, String name, int[] dailyRecords) {
            if (currentStudents >= maxStudents) {
                System.out.println("❌ Error: Class full ho chuki hai!");
                return;
            }
            if (dailyRecords.length != numDays) {
                System.out.println("❌ Error: " + numDays + " dino ka record dena zaroori hai.");
                return;
            }
            if (studentMap.containsKey(id)) {
                System.out.println("❌ Error: Student ID " + id + " pehle se maujood hai.");
                return;
            }

            // 2D Array mein attendance data save karna
            for (int j = 0; j < numDays; j++) {
                attendanceGrid[currentStudents][j] = dailyRecords[j];
            }

            // HashMap mein student ki location (row) save karna
            studentMap.put(id, new Student(id, name, currentStudents));
            currentStudents++;
            System.out.println("✅ Student '" + name + "' (ID: " + id + ") successfully add ho gaya!");
        }

        // Hashing: ID se turant record nikalna
        public void getAttendanceById(int id) {
            if (studentMap.containsKey(id)) {
                Student s = studentMap.get(id);
                int[] records = attendanceGrid[s.rowIndex];
                int presentCount = 0;
                for (int val : records) {
                    presentCount += val;
                }
                double percentage = ((double) presentCount / numDays) * 100;
                System.out.printf("ID: %d | Name: %s | Records: %s | Attendance: %.1f%%\n",
                        id, s.name, Arrays.toString(records), percentage);
            } else {
                System.out.println("❌ Student ID " + id + " nahi mila.");
            }
        }

        // 3. Searching: Naam se student dhoondhna (Linear Search)
        public void searchByName(String targetName) {
            for (Student s : studentMap.values()) {
                if (s.name.equalsIgnoreCase(targetName)) {
                    getAttendanceById(s.id);
                    return;
                }
            }
            System.out.println("❌ Student '" + targetName + "' nahi mila.");
        }

        // 4. Sorting: Bubble Sort se attendance percentage ke hisab se rank karna
        public void displaySortedReport() {
            if (currentStudents == 0) {
                System.out.println("Koi data available nahi hai.");
                return;
            }

            StudentReport[] reports = new StudentReport[currentStudents];
            int idx = 0;

            for (Student s : studentMap.values()) {
                int presentCount = 0;
                for (int val : attendanceGrid[s.rowIndex]) {
                    presentCount += val;
                }
                double percentage = ((double) presentCount / numDays) * 100;
                reports[idx++] = new StudentReport(s.id, s.name, percentage);
            }

            // Bubble Sort (Highest to Lowest)
            for (int i = 0; i < currentStudents - 1; i++) {
                for (int j = 0; j < currentStudents - i - 1; j++) {
                    if (reports[j].percentage < reports[j + 1].percentage) {
                        StudentReport temp = reports[j];
                        reports[j] = reports[j + 1];
                        reports[j + 1] = temp;
                    }
                }
            }

            System.out.println("\n📊 --- Sorted Attendance Report (Highest to Lowest) ---");
            for (StudentReport r : reports) {
                System.out.printf("ID: %-5d | Name: %-12s | Attendance: %.1f%%\n",
                        r.id, r.name, r.percentage);
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        // Max 50 students aur 5 dino ka attendance tracker
        AttendanceTracker tracker = new AttendanceTracker(50, 5);

        // Pehle se kuch sample data add kiya gaya hai taaki turant test ho sake
        tracker.addStudent(101, "Rahul", new int[]{1, 1, 1, 1, 1}); // 100%
        tracker.addStudent(102, "Priya", new int[]{1, 1, 1, 1, 0}); // 80%
        tracker.addStudent(103, "Amit",  new int[]{1, 0, 1, 1, 0}); // 60%
        tracker.addStudent(104, "Sneha", new int[]{0, 1, 0, 1, 0}); // 40%

        while (true) {
            System.out.println("\n==============================");
            System.out.println("   ATTENDANCE TRACKER MENU");
            System.out.println("==============================");
            System.out.println("1. Naya Student Add Karein");
            System.out.println("2. ID se Attendance Dekhein (Hashing)");
            System.out.println("3. Naam se Search Karein (Searching)");
            System.out.println("4. Sorted Report Dekhein (Sorting)");
            System.out.println("5. Exit");
            System.out.print("Apna option chunein (1-5): ");

            int choice;
            try {
                choice = Integer.parseInt(sc.nextLine());
            } catch (Exception e) {
                System.out.println("❌ Galat input! Kripya 1 se 5 ke beech number dalein.");
                continue;
            }

            if (choice == 1) {
                try {
                    System.out.print("Student ID dalein (e.g., 105): ");
                    int id = Integer.parseInt(sc.nextLine());

                    System.out.print("Student ka Naam dalein: ");
                    String name = sc.nextLine();

                    System.out.print("5 dino ki attendance dalein (1 = Present, 0 = Absent, space dekar, e.g., 1 1 0 1 1): ");
                    String[] parts = sc.nextLine().trim().split("\\s+");
                    int[] rec = new int[parts.length];
                    for (int i = 0; i < parts.length; i++) {
                        rec[i] = Integer.parseInt(parts[i]);
                    }
                    tracker.addStudent(id, name, rec);
                } catch (Exception e) {
                    System.out.println("❌ Galat input! Sahi format mein number dalein.");
                }
            } else if (choice == 2) {
                try {
                    System.out.print("Student ID dalein: ");
                    int id = Integer.parseInt(sc.nextLine());
                    tracker.getAttendanceById(id);
                } catch (Exception e) {
                    System.out.println("❌ Kripya valid ID number dalein.");
                }
            } else if (choice == 3) {
                System.out.print("Student ka Naam dalein: ");
                String name = sc.nextLine();
                tracker.searchByName(name);
            } else if (choice == 4) {
                tracker.displaySortedReport();
            } else if (choice == 5) {
                System.out.println("Program band ho raha hai. Dhanyavad!");
                break;
            } else {
                System.out.println("❌ Galat option! 1 se 5 ke beech chunein.");
            }
        }
        sc.close();
    }
}
