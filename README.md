# Handicap-Tracker
Java program that records golf rounds and calculates your golf handicap

/*
 * Golf Handicap and Round Tracker
 * Benjamin Arand
 * LaunchCode CS50x
 * June 2016
 * 
 * 
 * Application receives information input from user about golf round and course. 
 * It stores this information in SQL table and calculates the golfers handicap.
 */


package test;

import java.awt.BorderLayout;
import java.awt.EventQueue;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;
import javax.swing.JButton;
import java.awt.event.ActionListener;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.text.DecimalFormat;
import java.util.logging.Level;
import java.util.logging.Logger;
import java.awt.event.ActionEvent;
import javax.swing.JTextField;
import javax.swing.table.DefaultTableModel;
import javax.swing.JTable;
import javax.swing.JScrollPane;

public class SimpleGUI {

	private JFrame frame;
	private JTextField handicapscore;
	Connection con = null;
    PreparedStatement pstmt = null;
    Statement st = null;
    ResultSet rs = null;
    ResultSet rs2 = null;

    String url = "jdbc:mysql://localhost:3306";
    String user = "root";
    String password = "root";
	
	
	
	

	/**
	 * Launch the application.
	 */
	public static void main(String[] args) {
		EventQueue.invokeLater(new Runnable() {
			public void run() {
				try {
					SimpleGUI window = new SimpleGUI();
					window.frame.setVisible(true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	/**
	 * Create the application.
	 */
	public SimpleGUI() {
		initialize();
	}

	/**
	 * Initialize the contents of the frame.
	 */
	private void initialize() {
		frame = new JFrame();
		frame.setBounds(100, 100, 450, 300);
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		frame.getContentPane().setLayout(null);
		
		JLabel lblWelcomeToThe = new JLabel("Welcome to the handicap tracker!");
		lblWelcomeToThe.setBounds(119, 6, 247, 45);
		frame.getContentPane().add(lblWelcomeToThe);
		
		
	//Enter Round Button prompts user for course information and calls method getPositiveValue to
	//ensure non-negative number has been entered. Then calls method addHandicap to store info in SQL table.
		
		JButton btnEnterRound = new JButton("Enter Round");
		btnEnterRound.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				int slope = (int)getPositiveValue("Enter Course Slope", "Slope Dialog");
				double rating = getPositiveValue("Enter Course Rating", "Rating Dialog");
				int totalScore = (int)getPositiveValue("Enter total score", "Score Dialog");
				String course = (String)JOptionPane.showInputDialog(
						frame,
						"Enter Course Name",
						"Course Dialog", JOptionPane.PLAIN_MESSAGE,
						null,
						null,
						"");
				double HD = calculateHandicapDifferential(totalScore, rating, slope);
				addHandicap(HD, course, totalScore);	
			}
		});
		
		//Display Handicap button
		btnEnterRound.setBounds(150, 162, 144, 29);
		frame.getContentPane().add(btnEnterRound);
		JButton btnDisplayHandicap = new JButton("Display Handicap");
		btnDisplayHandicap.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				double handicap = displayHandicap();
				String hcap = df1.format(handicap);
				handicapscore.setText(hcap);
			}
		});
		
		
		btnDisplayHandicap.setBounds(150, 53, 144, 29);
		frame.getContentPane().add(btnDisplayHandicap);
	
		handicapscore = new JTextField();
		handicapscore.setBounds(178, 94, 78, 45);
		frame.getContentPane().add(handicapscore);
		handicapscore.setColumns(10);
		
		JButton btnNewButton = new JButton("Display Rounds");
		btnNewButton.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent e) {
				showTableData();
			}
		});
		
		
		btnNewButton.setBounds(150, 199, 144, 29);
		frame.getContentPane().add(btnNewButton);
		
	}
	
	
	/* 
	 * ---------------------------------METHODS------------------------------------------
	 * 
	 */
	
	
	//Formats double values to one decimal place.
	private static DecimalFormat df1 = new DecimalFormat(".#");
	
	
	//Converts user-inputed string to double and returns if positive, otherwise re-prompts.
	private double getPositiveValue(String prompt, String title)
	{
		String s = (String)JOptionPane.showInputDialog(
				frame,
				prompt,
				title, JOptionPane.PLAIN_MESSAGE,
				null,
				null,
				"");
		
		double value = Double.parseDouble(s);
		while (value < 0)
		{
			s = (String)JOptionPane.showInputDialog(
					frame,
					prompt + "(Please enter a positive value)",
					title, JOptionPane.PLAIN_MESSAGE,
					null,
					null,
					"");
			value = Double.parseDouble(s);	
		}
		return value;		
	}
	
	//Simple arithmetic method to calculate and return a handicap differential from a single round.
	public double calculateHandicapDifferential(int score, double rating, int slope)
	{
		double handicap = (score - rating) * 113 / slope;
		return handicap;
	}
	
	//Method that inserts new round into SQL table
	public void addHandicap(double handicap, String course, int score) 
	{
		
        String addRound = "INSERT INTO `Test Schema`.golfTable " + "VALUES (null, ?, ?, ?)";

        try {
            con = DriverManager.getConnection(url, user, password);
            pstmt = con.prepareStatement(addRound);
            
            pstmt.setDouble(1, handicap);
            pstmt.setString(2, course);
            pstmt.setInt(3, score);
            pstmt.executeUpdate();
            
            
        } catch (SQLException ex) {
            Logger lgr = Logger.getLogger(GolfMenu.class.getName());
            lgr.log(Level.SEVERE, ex.getMessage(), ex);

        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
                if (pstmt != null) {
                    pstmt.close();
                }
                if (con != null) {
                    con.close();
                }

            } catch (SQLException ex) {
                Logger lgr = Logger.getLogger(GolfMenu.class.getName());
                lgr.log(Level.WARNING, ex.getMessage(), ex);
            }
        }  
    }
	
	
	/* Complex Method that runs one of three possible handicap calculations depending on the amount of rounds
	 * stored in the SQL table. 
	 * 
	 * 
	 */
	public double displayHandicap()
	{
        int count = 0;
        double hcd = 0;
       
        //SQL statements
        String query = "SELECT COUNT(id) FROM `Test Schema`.golfTable";
        String min = "SELECT MIN(handicap) FROM `Test Schema`.golfTable";
        String handiMid = "SELECT handicap FROM `Test Schema`.golfTable ORDER BY handicap ASC LIMIT 5";
        String handiBig = "SELECT handicap FROM `Test Schema`.golfTable ORDER BY handicap ASC LIMIT 10";
        		

        try {
            con = DriverManager.getConnection(url, user, password);
            st = con.createStatement();
            rs = st.executeQuery(query); 
            
            if (rs.next()) {
            	count = rs.getInt(1);
            	//System.out.println(count); 
            }   
        }
        catch (SQLException ex) {
            Logger lgr = Logger.getLogger(tester.class.getName());
            lgr.log(Level.SEVERE, ex.getMessage(), ex);

        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
                if (st != null) {
                    st.close();
                }
                if (con != null) {
                    con.close();
                }

            } catch (SQLException ex) {
                Logger lgr = Logger.getLogger(tester.class.getName());
                lgr.log(Level.WARNING, ex.getMessage(), ex);
            }
        } 
        
        //If less then 5 rounds have been entered, handicap can not be calculated.
        if (count < 5)
        {
        	System.out.println("Minimum 5 golf rounds required to calculate handicap");
        	return 1;
        }
        
        //Calculation for rounds between 5 - 10
        else if (count >= 5 && count <= 10)
        {
	        try {
	            con = DriverManager.getConnection(url, user, password);
	            st = con.createStatement();
	            rs2 = st.executeQuery(min); 
	            
	            //Takes smallest handicap differential from table and multiplies it by .96
	            //to calculate handicap.
	            if (rs2.next()) {
	            	hcd = rs2.getDouble(1);
	            	hcd = hcd * .96;	
	            }   
	        }
	        catch (SQLException ex) {
	            Logger lgr = Logger.getLogger(tester.class.getName());
	            lgr.log(Level.SEVERE, ex.getMessage(), ex);
	
	        } finally {
	            try {
	                if (rs2 != null) {
	                    rs.close();
	                }
	                if (st != null) {
	                    st.close();
	                }
	                if (con != null) {
	                    con.close();
	                }
	
	            } catch (SQLException ex) {
	                Logger lgr = Logger.getLogger(tester.class.getName());
	                lgr.log(Level.WARNING, ex.getMessage(), ex);
	            }
	        }
	        return hcd;
        }
        
      //Calculation for rounds between 11 - 19
        else if (count >= 11 && count <= 19)
        {
	        try {
	            con = DriverManager.getConnection(url, user, password);
	            st = con.createStatement();
	            rs2 = st.executeQuery(handiMid); 
	            
	            while (rs2.next()) {
		            hcd = hcd + rs2.getDouble(1);   	
	            }
	            //Takes the 5 lowest handicap differentials, averages them, and multiplies by .96
	            //to get handicap
	            hcd = hcd / 5;
	            hcd = hcd * .96;  
	        }
	        
	        catch (SQLException ex) {
	            Logger lgr = Logger.getLogger(tester.class.getName());
	            lgr.log(Level.SEVERE, ex.getMessage(), ex);
	
	        } finally {
	            try {
	                if (rs2 != null) {
	                    rs.close();
	                }
	                if (st != null) {
	                    st.close();
	                }
	                if (con != null) {
	                    con.close();
	                }
	
	            } catch (SQLException ex) {
	                Logger lgr = Logger.getLogger(tester.class.getName());
	                lgr.log(Level.WARNING, ex.getMessage(), ex);
	            }
	        }
	        return hcd;
        }
        
        //Handicap calculation for 20 or more rounds
        else if (count >= 20)
        {
	        try {
	            con = DriverManager.getConnection(url, user, password);
	            st = con.createStatement();
	            rs2 = st.executeQuery(handiBig); 
	            
	            //Creates sum of 10 lowest handicap differentials
	            while (rs2.next()) {
		            hcd = hcd + rs2.getDouble(1);   	
	            }
	            //Averages the differentials and multiplies by .96 to get handicap
	            hcd = hcd / 10;
	            hcd = hcd * .96;
	        }
	        
	        catch (SQLException ex) {
	            Logger lgr = Logger.getLogger(tester.class.getName());
	            lgr.log(Level.SEVERE, ex.getMessage(), ex);
	
	        } finally {
	            try {
	                if (rs2 != null) {
	                    rs.close();
	                }
	                if (st != null) {
	                    st.close();
	                }
	                if (con != null) {
	                    con.close();
	                }
	
	            } catch (SQLException ex) {
	                Logger lgr = Logger.getLogger(tester.class.getName());
	                lgr.log(Level.WARNING, ex.getMessage(), ex);
	            }
	        }
        }
        return hcd;    
    }
	
	//Method to create a new window and display all golf rounds in SQL table
	public void showTableData() {
		
		frame = new JFrame("Database Search Result");
		frame.setBounds(100, 100, 450, 300);
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		
		String[] columnNames = {"id", "handicap", "course", "score"};
		
		PreparedStatement pst;
		
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.getContentPane().setLayout(new BorderLayout());

        DefaultTableModel model = new DefaultTableModel();
        model.setColumnIdentifiers(columnNames);

        JTable table = new JTable();
        table.setModel(model);
        table.setAutoResizeMode(JTable.AUTO_RESIZE_ALL_COLUMNS);
        table.setFillsViewportHeight(true);
        JScrollPane scroll = new JScrollPane(table);
        scroll.setHorizontalScrollBarPolicy(
                JScrollPane.HORIZONTAL_SCROLLBAR_AS_NEEDED);
        scroll.setVerticalScrollBarPolicy(
                JScrollPane.VERTICAL_SCROLLBAR_AS_NEEDED);
        
        String id = "";
        String hcd = "";
        String course = "";
        String score = "";
 
        try {
        	Connection con = DriverManager.getConnection(url, user, password);
            pst = con.prepareStatement("SELECT * FROM `Test Schema`.golfTable");
            ResultSet rs = pst.executeQuery();
            int i = 0;
            
            while (rs.next()) {
                id = rs.getString("id");
                hcd = rs.getString("handicap");
                course = rs.getString("course");
                score = rs.getString("score");
                model.addRow(new Object[]{id, hcd, course, score});
                i++;
            }
            if (i < 1) {
                JOptionPane.showMessageDialog(null, "No Record Found", "Error", JOptionPane.ERROR_MESSAGE);
            }
            
            /*
            if (i == 1) {
                System.out.println(i + " Record Found");
            } 
            else {
                System.out.println(i + " Records Found");
            }
            */
            
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(null, ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
        frame.getContentPane().add(scroll);
        frame.setVisible(true);
        frame.setSize(400, 300);
        initialize();
    }
	
}	
