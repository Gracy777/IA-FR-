import streamlit as st
from face_analyzer import FaceAnalyzer
import io

def main():
    st.set_page_config(
        page_title="Face Analysis Tool",
        page_icon="👤",
        layout="wide"
    )

    st.title("Face Analysis Tool")
    st.write("Upload two images to compare faces and analyze their characteristics")

    # Initialize FaceAnalyzer
    analyzer = FaceAnalyzer()

    # Create two columns for image upload
    col1, col2 = st.columns(2)

    with col1:
        st.subheader("First Image")
        image1 = st.file_uploader("Upload first image", type=['jpg', 'jpeg', 'png'], key="image1")

    with col2:
        st.subheader("Second Image")
        image2 = st.file_uploader("Upload second image", type=['jpg', 'jpeg', 'png'], key="image2")

    if image1 is not None and image2 is not None:
        # Validate images
        valid1, msg1 = analyzer.validate_image(image1.read())
        image1.seek(0)  # Reset file pointer
        valid2, msg2 = analyzer.validate_image(image2.read())
        image2.seek(0)  # Reset file pointer

        if not valid1 or not valid2:
            st.error(f"Image validation failed: {msg1 if not valid1 else msg2}")
            return

        # Show progress
        with st.spinner("Analyzing faces..."):
            # Analyze faces
            result = analyzer.analyze_faces(image1.read(), image2.read())

        if result['success']:
            # Display results
            st.success("Analysis Complete!")
            
            # Create metrics
            col1, col2, col3, col4 = st.columns(4)
            
            with col1:
                st.metric("Face Similarity", f"{result['similarity_score']:.2f}%")
            with col2:
                st.metric("Blur Difference", f"{result['blur_difference']:.2f}")
            with col3:
                st.metric("Rotation Difference", f"{result['rotation_difference']:.2f}°")
            with col4:
                st.metric("Noise Difference", f"{result['noise_difference']:.2f}")

            # Display detailed analysis
            with st.expander("Detailed Analysis"):
                st.write("Face Similarity Score:", f"{result['similarity_score']:.2f}%")
                st.write("This score indicates how similar the faces are, with 100% being an exact match.")
                
                st.write("\nImage Quality Analysis:")
                st.write(f"- Blur Difference: {result['blur_difference']:.2f}")
                st.write(f"- Rotation Difference: {result['rotation_difference']:.2f}°")
                st.write(f"- Noise Difference: {result['noise_difference']:.2f}")
                
                st.write("\nInterpretation:")
                st.write("- Lower blur difference indicates similar image clarity")
                st.write("- Lower rotation difference indicates similar face angles")
                st.write("- Lower noise difference indicates similar image quality")

        else:
            st.error(f"Analysis failed: {result['error']}")

    # Add usage instructions
    with st.sidebar:
        st.header("Instructions")
        st.write("""
        1. Upload two images containing faces
        2. Images should be in JPG or PNG format
        3. Maximum file size: 5MB
        4. Each image should contain at least one clear face
        5. Wait for the analysis to complete
        """)

        st.header("About")
        st.write("""
        This tool analyzes and compares faces in images using:
        - Face detection and recognition
        - Blur detection
        - Rotation analysis
        - Noise level comparison
        """)

if __name__ == "__main__":
    main()
